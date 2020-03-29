## 1. catalog和SimpleFeatureType创建
```
./geomesa-hbase create-schema -c 'findtest:test' -s "bizid:String:index=true,biztime:Date,*bizpoint:Point:srid=4326;geomesa.attr.splits='11',geomesa.indices='attr:8:3:bizid:geom:biztime',geomesa.z3.interval='day'" -f 'point' --dtg 'biztime' --compression 'snappy'

```

## 2. 批量导入

准备数据文件

```
data.csv
693779,118.733589483333000,32.066745283333300,2019-11-11T00:00:00.000Z
672623,131.133605000000000,47.097039000000000,2019-11-12T00:00:00.000Z
```

准备convert映射文件c.convert
<br>
这里的convert文件格式有很多种，可以解析json，xml，csv等。[具体详情
][2]<br>
注意这里的[transform函数有][3]

```
geomesa.converters.point = {
    type = "delimited-text",
    format = "CSV",
    id-field = "md5(stringToBytes($0))",
    fields = [
        { name = "bizid",    transform = "$1::string"    }
        { name = "lon",    transform = "$2::double"    }
        { name = "lat",    transform = "$3::double"    }
        { name = "bizpoint",    transform = "point($lon, $lat)"    }
        { name = "biztime",    transform = "dateTime($4)"    }
    ]
    user-data = {
    // note: keys will be treated as strings and should not be quoted
    my.user.key = "$phrase"
  }
}

```
执行导入命令

```

./geomesa-hbase ingest \
-c 'biznamespace:bzitable' -f 'biz_sftname' \
-C c.convert  \
data.csv 

INFO  Schema 'biz_sftname' exists
INFO  Running ingestion in local mode
INFO  Ingesting 1 file with 1 thread
[============================================================] 100% complete 2 ingested 0 failed in 00:00:01
INFO  Local ingestion complete in 00:00:01
INFO  Ingested 2 features and failed to ingest 1 feature for file: data.csv

```
## 查询导出方法

```
./geomesa-hbase export -c 'biznamespace:bzitable' -f 'biz_sftname' -a 'bizid, biztime, bizpoint' -q 'bizid =251184 AND biztime DURING 2019-11-11T00:00:00.000Z/2019-11-20T00:00:00.003Z' -F csv -m 10 -o a.csv
 
 
-c, --catalog * The catalog table containing schema metadata
-f, --feature-name *    The name of the schema
-q, --cql   CQL filter to select features to export
-a, --attributes    Specific attributes to export
-m, --max-features  Limit the number of features exported
-F, --output-format Output format used for export
-o, --output    Output to a file instead of standard out
--hints Query hints used to modify the query
--index Specific index used to run the query
--no-header Don’t export the type header, for CSV and TSV formats
--gzip  Level of gzip compression to use for output, from 1-9

```
 
## 查询计划
```
geomesa-hbase explain -c 'biznamespace:bzitable' -f 'biz_sftname' -a 'bizid, biztime, bizpoint' -q 'bizid =251184 AND biztime DURING 2019-11-11T00:00:00.000Z/2019-11-20T00:00:00.003Z' 
 
 
-c, --catalog * The catalog table containing schema metadata
-f, --feature-name *    The name of the schema
-q, --cql * CQL filter to select features to export
-a, --attributes    Specific attributes to export
--hints Query hints used to modify the query
--index Specific index used to run the query

```
 
## 更改schema 命令
<br>更多操作见[schema命令][4]<br>
### 删除 catalog
```
geomesa-hbase delete-catalog -c 'biznamespace:bzitable'  

```
### 删除 schema
```
geomesa-hbase remove-schema -c 'biznamespace:bzitable' -f 'biz_sftname'
```

### 看schema结构

```
geomesa-hbase describe-schema -c 'biznamespace:bzitable' -f 'biz_sftname' 

```


## 附录
### 其他种类的
```

geomesa.converters.ais-data = {
    type = "delimited-text",
    format = "CSV",
    id-field = "toString($arkposid)",
    fields = [
        { name = "arkposid",    transform = "$1::long"    }
        { name = "mmsi",    transform = "$2::int"    }
        { name = "navigationalstatus",    transform = "$3::int"    }
        { name = "lon",    transform = "$4::double"    }
        { name = "lat",    transform = "$5::double"    }
        { name = "coords",    transform = "point($lon, $lat)"    }
        { name = "sog",    transform = "$6::double"    }
        { name = "cog",    transform = "$7::double"    }
    	{ name = "rot",    transform = "$8::double"    }
        { name = "heading",    transform = "$9::int"    }
    	{ name = "acquisitiontime",    transform = "toString($10)"    }
    	{ name = "iptype",    transform = "$11::int"    }
    ]
}
```
 
[1]: https://www.geomesa.org/documentation/user/convert/usage_tools.html
[2]: https://www.geomesa.org/documentation/user/convert/delimited_text.html
[3]: https://www.geomesa.org/documentation/user/convert/function_usage.html
[4]: https://www.geomesa.org/documentation/user/cli/schemas.html#remove-schema