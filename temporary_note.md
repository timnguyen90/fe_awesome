docker pull mongo
docker run -d -p 27017:27017 --name shopping-mongo mongo
docker logs -f shopping-mongo
docker exec -it shopping-mongo /bin/bash
mongosh
show dbs
use CatalogDb (create if not exist)
db.createCollection('Products')
show collections
