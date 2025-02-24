mongo命令行

```
mongo --username admin --password MVFGAA190y2fbb55sZ1360
show dbs
use vsa_manage
show collections | db.getCollectionNames()
db.tool.find({tool:"w"})
db.tool.update({tool:"www"},{$set:{tool:"wwww"}})
db.tool.remove({ key: "value" })
```

