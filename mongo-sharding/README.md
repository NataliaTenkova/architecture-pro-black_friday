## Инициализация на сервере конфигурации
docker exec -it configSrv mongosh --port 27017

> rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
> exit();

## Инициализация шардов
docker exec -it shard1 mongosh --port 27018
> rs.initiate(
    {
      _id : "shard1",
      members: [
        { _id : 0, host : "shard1:27018" },
       // { _id : 1, host : "shard2:27019" }
      ]
    }
);
> exit();

docker exec -it shard2 mongosh --port 27019
> rs.initiate(
    {
      _id : "shard2",
      members: [
       // { _id : 0, host : "shard1:27018" },
        { _id : 1, host : "shard2:27019" }
      ]
    }
  );
> exit();

## Инициализация роутера и наполнение данными
docker exec -it mongos_router mongosh --port 27020

> sh.addShard( "shard1/shard1:27018");
> sh.addShard( "shard2/shard2:27019");

> sh.enableSharding("somedb");
> sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

> use somedb

> for(var i = 0; i < 1000; i++) db.helloDoc.insertOne({age:i, name:"ly"+i});
> db.helloDoc.countDocuments();
> exit();

## Проверка на шардах
docker exec -it shard1 mongosh --port 27018
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();

docker exec -it shard2 mongosh --port 27019
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();