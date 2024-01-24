# first, init ssl certificate
```
docker-compose -f init.yaml up
```
> it will autocreate ssl certificate include pem and jks.
> if you want to update your password plz update generate.sh line:5 
   and update docker-compose.yaml line:23

> sasl client password: line 18
# start cluster
```
docker-compose up -d 
```