七层网络协议中，接触比较多主要是应用层协议比如： HTTP，FTP，MQTT等。 对于应用层协议来说，底层一般可以基于TCP或者UDP协议来进行实现。
但是实际工作中可能会遇到很多自定义的协议，这些协议往往基于TCP。比如 PG V3协议， Mysql协议，Dubbo协议等。 后面几篇文章我们会用netty实现server，来逐一对这些协议进行破解。
-   [PG V3 协议的 官方文档][1]
-   [Mysql 协议官方文档][2]

下面我们先讲解如实如何使用Netty 来破解pg协议。 [项目源码][5]

## PG V3 协议

### [协议解析处理流程][3], [协议内容][4]

-   Start Up : 每次开启一个数据查询session，前端都会发送一个startup message，包含用户名，密码，连接参数。server端解析后，来初始化session的configuration信息。 用到的message如下

    -   AuthenticationOk
        -   Byte1('R')
            Identifies the message as an authentication request.
            
            Int32(8)
            Length of message contents in bytes, including self.
            
            Int32(0)
            Specifies that the authentication was successful.
    -   AuthenticationMD5Password
        -   Byte1('R')
            Identifies the message as an authentication request.
            
            Int32(12)
            Length of message contents in bytes, including self.
            
            Int32(5)
            Specifies that an MD5-encrypted password is required.
            
            Byte4
            The salt to use when encrypting the password.
    -   NegotiateProtocolVersion
        -   Byte1('v')
            Identifies the message as a protocol version negotiation message.
            
            Int32
            Length of message contents in bytes, including self.
            
            Int32
            Newest minor protocol version supported by the server for the major protocol version requested by the client.
            
            Int32
            Number of protocol options not recognized by the server.
            
            Then, for protocol option not recognized by the server, there is the following:
            
            String
            The option name.
    -   BackendKeyData
        -   Byte1('K')
            Identifies the message as cancellation key data. The frontend must save these values if it wishes to be able to issue CancelRequest messages later.
            
            Int32(12)
            Length of message contents in bytes, including self.
            
            Int32
            The process ID of this backend.
            
            Int32
            The secret key of this backend.
    -   ParameterStatus
        -   Byte1('S')
            Identifies the message as a run-time parameter status report.
            
            Int32
            Length of message contents in bytes, including self.
            
            String
            The name of the run-time parameter being reported.
            
            String
            The current value of the parameter.

-   SimpleQuery: 一种文本作为传输内容的简单协议。
    -   CommandComplete
        -   Byte1('C')
            Identifies the message as a command-completed response.
            
            Int32
            Length of message contents in bytes, including self.
            
            String
            The command tag. This is usually a single word that identifies which SQL command was completed.
            
            For an INSERT command, the tag is INSERT oid rows, where rows is the number of rows inserted. oid is the object ID of the inserted row if rows is 1 and the target table has OIDs; otherwise oid is 0.
            
            For a DELETE command, the tag is DELETE rows where rows is the number of rows deleted.
            
            For an UPDATE command, the tag is UPDATE rows where rows is the number of rows updated.
            
            For a SELECT or CREATE TABLE AS command, the tag is SELECT rows where rows is the number of rows retrieved.
            
            For a MOVE command, the tag is MOVE rows where rows is the number of rows the cursor's position has been changed by.
            
            For a FETCH command, the tag is FETCH rows where rows is the number of rows that have been retrieved from the cursor.
            
            For a COPY command, the tag is COPY rows where rows is the number of rows copied. (Note: the row count appears only in PostgreSQL 8.2 and later.)
    -   RowDescription
        -   Byte1('T')
            Identifies the message as a row description.
            
            Int32
            Length of message contents in bytes, including self.
            
            Int16
            Specifies the number of fields in a row (can be zero).
            
            Then, for each field, there is the following:
            
            String
            The field name.
            
            Int32
            If the field can be identified as a column of a specific table, the object ID of the table; otherwise zero.
            
            Int16
            If the field can be identified as a column of a specific table, the attribute number of the column; otherwise zero.
            
            Int32
            The object ID of the field's data type.
            
            Int16
            The data type size (see pg_type.typlen). Note that negative values denote variable-width types.
            
            Int32
            The type modifier (see pg_attribute.atttypmod). The meaning of the modifier is type-specific.
            
            Int16
            The format code being used for the field. Currently will be zero (text) or one (binary). In a RowDescription returned from the statement variant of Describe, the format code is not yet known and will always be zero.
    -   DataRow
        -   Byte1('D')
            Identifies the message as a data row.
            
            Int32
            Length of message contents in bytes, including self.
            
            Int16
            The number of column values that follow (possibly zero).
            
            Next, the following pair of fields appear for each column:
            
            Int32
            The length of the column value, in bytes (this count does not include itself). Can be zero. As a special case, -1 indicates a NULL column value. No value bytes follow in the NULL case.
            
            Byten
            The value of the column, in the format indicated by the associated format code. n is the above length.
    -   EmptyQueryResponse
        -   Byte1('I')
            Identifies the message as a response to an empty query string. (This substitutes for CommandComplete.)
            
            Int32(4)
            Length of message contents in bytes, including self.
            
-   Extended Query：
    -   Parse message
    -   ParseComplete    
-   Function Call
    -   FunctionCallResponse
        
-   所有过程都会用到的message：    
    -   ErrorResponse
        -   Byte1('E')
            Identifies the message as an error.
            
            Int32
            Length of message contents in bytes, including self.
            
            The message body consists of one or more identified fields, followed by a zero byte as a terminator. Fields can appear in any order. For each field there is the following:
            
            Byte1
            A code identifying the field type; if zero, this is the message terminator and no string follows. The presently defined field types are listed in Section 49.6. Since more field types might be added in future, frontends should silently ignore fields of unrecognized type.
            
            String
            The field value.
    -   ReadyForQuery
        -   Byte1('Z')
            Identifies the message type. ReadyForQuery is sent whenever the backend is ready for a new query cycle.
            
            Int32(5)
            Length of message contents in bytes, including self.
            
            Byte1
            Current backend transaction status indicator. Possible values are 'I' if idle (not in a transaction block); 'T' if in a transaction block; or 'E' if in a failed transaction block (queries will be rejected until block is ended).
    -   NoticeResponse
        -   Byte1('N')
         Identifies the message as a notice.
         
         Int32
         Length of message contents in bytes, including self.
         
         The message body consists of one or more identified fields, followed by a zero byte as a terminator. Fields can appear in any order. For each field there is the following:
         
         Byte1
         A code identifying the field type; if zero, this is the message terminator and no string follows. The presently defined field types are listed in Section 49.6. Since more field types might be added in future, frontends should silently ignore fields of unrecognized type.
         
         String
         The field value.
    
[1]: https://www.postgresql.org/docs/9.4/protocol-overview.html
[2]: https://dev.mysql.com/doc/internals/en/client-server-protocol.html
[3]: https://www.postgresql.org/docs/9.4/protocol-flow.html
[4]: https://www.postgresql.org/docs/9.4/protocol-message-formats.html
[5]: https://github.com/moheqionglin/spring-demo/tree/master/other-project/netty-app/src/main/java/com/moheqionglin/pgv3server