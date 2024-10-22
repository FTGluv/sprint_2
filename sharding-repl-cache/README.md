docker compose up -d

# Необходимо добавить конфигурацию роутера
docker exec -it configSrv mongosh --port 27017

rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);

exit(); 

# Необходимо сконфигурировать шарды
docker exec -it shard1 mongosh --port 27018

rs.initiate(
    {
      _id : "shard1",
      members: [
        { _id : 0, host : "shard1:27018" }
      ]
    }
);

exit();

docker exec -it shard2 mongosh --port 27019

rs.initiate(
    {
      _id : "shard2",
      members: [
        { _id : 1, host : "shard2:27019" }
      ]
    }
  );

  exit(); 

  # Теперь конфигурируем роутер
  docker exec -it mongos_router mongosh --port 27020

  sh.addShard( "shard1/shard1:27018");

  sh.addShard( "shard2/shard2:27019");

  sh.enableSharding("somedb");

  sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

# Заполняем базу тестовыми данными
  use somedb

  for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

# Проверка

docker exec -it shard1 mongosh --port 27018
use somedb;
db.helloDoc.countDocuments();
exit(); 

docker exec -it shard2 mongosh --port 27019
use somedb;
db.helloDoc.countDocuments();
exit(); 


# Testing the use of cache
http://localhost:8080/helloDoc/users

# Configuration of shard 1 replica 1
docker exec -it shard1 mongosh --port 27018

rs.initiate({_id: "shard1", members: [
{_id: 0, host: "shard1:27018"},
{_id: 1, host: "shard1_repl1:27022"},
{_id: 2, host: "shard1_repl2:27023"}
]});

exit;

# Configuration of shard 1 replica 2
docker exec -it shard2 mongosh --port 27019

rs.initiate({_id: "shard2", members: [
{_id: 0, host: "shard2:27019"},
{_id: 1, host: "shard2_repl1:27024"},
{_id: 2, host: "shard2_repl2:27025"}
]});

exit;