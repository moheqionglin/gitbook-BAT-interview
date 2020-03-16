```
https://www.postgresql.org/docs/9.4/protocol-flow.html#PROTOCOL-ASYNC

```

## 客户端连接和认证过程

```
sendStartupPacket(newStream, paramList);

// Do authentication (until AuthenticationOk).
doAuthentication(newStream, hostSpec.getHost(), user, info);

    readStartupMessages();
```

```
org.postgresql.core.v3.ConnectionFactoryImpl.sendStartupPacket(newStream, paramList);

	int length = 4 + 4;
    byte[][] encodedParams = new byte[params.size() * 2][];
    for (int i = 0; i < params.size(); ++i) {
      encodedParams[i * 2] = params.get(i)[0].getBytes("UTF-8");
      encodedParams[i * 2 + 1] = params.get(i)[1].getBytes("UTF-8");
      length += encodedParams[i * 2].length + 1 + encodedParams[i * 2 + 1].length + 1;
    }

    length += 1; // Terminating \0

    // Send the startup message.
    pgStream.sendInteger4(length);
    pgStream.sendInteger2(3); // protocol major
    pgStream.sendInteger2(0); // protocol minor
    for (byte[] encodedParam : encodedParams) {
      pgStream.send(encodedParam);
      pgStream.sendChar(0);
    }

    pgStream.sendChar(0);
    pgStream.flush();
    
```

```
org.postgresql.core.v3.ConnectionFactoryImpl#doAuthentication
authloop: while (true) {
        int beresp = pgStream.receiveChar();

        switch (beresp) {
          case 'E':
            // An error occurred, so pass the error message to the
            // user.
            //
            // The most common one to be thrown here is:
            // "User authentication failed"
            //
            int l_elen = pgStream.receiveInteger4();
			  String error = pgStream.receiveErrorString(l_elen - 4)

          case 'R': // [1. 第一步发送R请求]
            // Authentication request.
            // Get the message length
            int l_msgLen = pgStream.receiveInteger4();

            // Get the type of request
            int areq = pgStream.receiveInteger4();

            // Process the request.
            switch (areq) {
              case AUTH_REQ_MD5: {//5
                byte[] md5Salt = pgStream.receive(4);
                if (LOGGER.isLoggable(Level.FINEST)) {
                  LOGGER.log(Level.FINEST, " <=BE AuthenticationReqMD5(salt={0})", Utils.toHexString(md5Salt));
                }

                if (password == null) {
                  throw new PSQLException(
                      GT.tr(
                          "The server requested password-based authentication, but no password was provided."),
                      PSQLState.CONNECTION_REJECTED);
                }

                byte[] digest =
                    MD5Digest.encode(user.getBytes("UTF-8"), password.getBytes("UTF-8"), md5Salt);

                if (LOGGER.isLoggable(Level.FINEST)) {
                  LOGGER.log(Level.FINEST, " FE=> Password(md5digest={0})", new String(digest, "US-ASCII"));
                }

                pgStream.sendChar('p');
                pgStream.sendInteger4(4 + digest.length + 1);
                pgStream.send(digest);
                pgStream.sendChar(0);
                pgStream.flush();

                break;
              }

              case AUTH_REQ_PASSWORD: {//3
                LOGGER.log(Level.FINEST, "<=BE AuthenticationReqPassword");
                LOGGER.log(Level.FINEST, " FE=> Password(password=<not shown>)");

                if (password == null) {
                  throw new PSQLException(
                      GT.tr(
                          "The server requested password-based authentication, but no password was provided."),
                      PSQLState.CONNECTION_REJECTED);
                }

                byte[] encodedPassword = password.getBytes("UTF-8");

                pgStream.sendChar('p');
                pgStream.sendInteger4(4 + encodedPassword.length + 1);
                pgStream.send(encodedPassword);
                pgStream.sendChar(0);
                pgStream.flush();

                break;
              }

              case AUTH_REQ_GSS:
              case AUTH_REQ_SSPI:
                /*
                 * Use GSSAPI if requested on all platforms, via JSSE.
                 *
                 * For SSPI auth requests, if we're on Windows attempt native SSPI authentication if
                 * available, and if not disabled by setting a kerberosServerName. On other
                 * platforms, attempt JSSE GSSAPI negotiation with the SSPI server.
                 *
                 * Note that this is slightly different to libpq, which uses SSPI for GSSAPI where
                 * supported. We prefer to use the existing Java JSSE Kerberos support rather than
                 * going to native (via JNA) calls where possible, so that JSSE system properties
                 * etc continue to work normally.
                 *
                 * Note that while SSPI is often Kerberos-based there's no guarantee it will be; it
                 * may be NTLM or anything else. If the client responds to an SSPI request via
                 * GSSAPI and the other end isn't using Kerberos for SSPI then authentication will
                 * fail.
                 */
                final String gsslib = PGProperty.GSS_LIB.get(info);
                final boolean usespnego = PGProperty.USE_SPNEGO.getBoolean(info);

                boolean useSSPI = false;

                /*
                 * Use SSPI if we're in auto mode on windows and have a request for SSPI auth, or if
                 * it's forced. Otherwise use gssapi. If the user has specified a Kerberos server
                 * name we'll always use JSSE GSSAPI.
                 */
                if (gsslib.equals("gssapi")) {
                  LOGGER.log(Level.FINE, "Using JSSE GSSAPI, param gsslib=gssapi");
                } else if (areq == AUTH_REQ_GSS && !gsslib.equals("sspi")) {
                  LOGGER.log(Level.FINE,
                      "Using JSSE GSSAPI, gssapi requested by server and gsslib=sspi not forced");
                } else {
                  /* Determine if SSPI is supported by the client */
                  sspiClient = createSSPI(pgStream, PGProperty.SSPI_SERVICE_CLASS.get(info),
                      /* Use negotiation for SSPI, or if explicitly requested for GSS */
                      areq == AUTH_REQ_SSPI || (areq == AUTH_REQ_GSS && usespnego));

                  useSSPI = sspiClient.isSSPISupported();
                  LOGGER.log(Level.FINE, "SSPI support detected: {0}", useSSPI);

                  if (!useSSPI) {
                    /* No need to dispose() if no SSPI used */
                    sspiClient = null;

                    if (gsslib.equals("sspi")) {
                      throw new PSQLException(
                          "SSPI forced with gsslib=sspi, but SSPI not available; set loglevel=2 for details",
                          PSQLState.CONNECTION_UNABLE_TO_CONNECT);
                    }
                  }

                  if (LOGGER.isLoggable(Level.FINE)) {
                    LOGGER.log(Level.FINE, "Using SSPI: {0}, gsslib={1} and SSPI support detected", new Object[]{useSSPI, gsslib});
                  }
                }

                if (useSSPI) {
                  /* SSPI requested and detected as available */
                  sspiClient.startSSPI();
                } else {
                  /* Use JGSS's GSSAPI for this request */
                  org.postgresql.gss.MakeGSS.authenticate(pgStream, host, user, password,
                      PGProperty.JAAS_APPLICATION_NAME.get(info),
                      PGProperty.KERBEROS_SERVER_NAME.get(info), usespnego,
                      PGProperty.JAAS_LOGIN.getBoolean(info));
                }
                break;

              case AUTH_REQ_GSS_CONTINUE:
                /*
                 * Only called for SSPI, as GSS is handled by an inner loop in MakeGSS.
                 */
                sspiClient.continueSSPI(l_msgLen - 8);
                break;

              case AUTH_REQ_SASL:
                LOGGER.log(Level.FINEST, " <=BE AuthenticationSASL");

                //JCP! if mvn.project.property.postgresql.jdbc.spec >= "JDBC4.2"
                scramAuthenticator = new org.postgresql.jre8.sasl.ScramAuthenticator(user, password, pgStream);
                scramAuthenticator.processServerMechanismsAndInit();
                scramAuthenticator.sendScramClientFirstMessage();
                //JCP! else
//JCP>                 if (true) {
//JCP>                   throw new PSQLException(GT.tr(
//JCP>                           "SCRAM authentication is not supported by this driver. You need JDK >= 8 and pgjdbc >= 42.2.0 (not \".jre\" versions)",
//JCP>                           areq), PSQLState.CONNECTION_REJECTED);
//JCP>                 }
                //JCP! endif
                break;

              //JCP! if mvn.project.property.postgresql.jdbc.spec >= "JDBC4.2"
              case AUTH_REQ_SASL_CONTINUE:
                scramAuthenticator.processServerFirstMessage(l_msgLen - 4 - 4);
                break;

              case AUTH_REQ_SASL_FINAL:
                scramAuthenticator.verifyServerSignature(l_msgLen - 4 - 4);
                break;
              //JCP! endif

              case AUTH_REQ_OK:
                /* Cleanup after successful authentication */
                LOGGER.log(Level.FINEST, " <=BE AuthenticationOk");
                break authloop; // We're done.

              default:
                LOGGER.log(Level.FINEST, " <=BE AuthenticationReq (unsupported type {0})", areq);
                throw new PSQLException(GT.tr(
                    "The authentication type {0} is not supported. Check that you have configured the pg_hba.conf file to include the client''s IP address or subnet, and that it is using an authentication scheme supported by the driver.",
                    areq), PSQLState.CONNECTION_REJECTED);
            }

            break;

          default:
            throw new PSQLException(GT.tr("Protocol error.  Session setup failed."),
                PSQLState.PROTOCOL_VIOLATION);
        }
      }
```

```
for (int i = 0; i < 1000; i++) {
      int beresp = pgStream.receiveChar();
      switch (beresp) {
        case 'Z':
          receiveRFQ();
          // Ready For Query; we're done.
          return;

        case 'K':
          // BackendKeyData
          int l_msgLen = pgStream.receiveInteger4();
          if (l_msgLen != 12) {
            throw new PSQLException(GT.tr("Protocol error.  Session setup failed."),
                PSQLState.PROTOCOL_VIOLATION);
          }

          int pid = pgStream.receiveInteger4();
          int ckey = pgStream.receiveInteger4();

          if (LOGGER.isLoggable(Level.FINEST)) {
            LOGGER.log(Level.FINEST, " <=BE BackendKeyData(pid={0},ckey={1})", new Object[]{pid, ckey});
          }

          setBackendKeyData(pid, ckey);
          break;

        case 'E':
          // Error
          throw receiveErrorResponse();

        case 'N':
          // Warning
          addWarning(receiveNoticeResponse());
          break;

        case 'S':
          // ParameterStatus
          receiveParameterStatus();

          break;

        default:
          if (LOGGER.isLoggable(Level.FINEST)) {
            LOGGER.log(Level.FINEST, "  invalid message type={0}", (char) beresp);
          }
          throw new PSQLException(GT.tr("Protocol error.  Session setup failed."),
              PSQLState.PROTOCOL_VIOLATION);
      }
    }
```
