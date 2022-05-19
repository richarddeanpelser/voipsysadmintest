### Wordpress
Wordpress can be accessed via http://91.151.1.10:8000/
**Note:** Since this server is publically accessible I'll leave the wordpress container to listen for external traffic on port 8000 instead of 80.
To change this to 80, simply edit the docker-compose.yml file and change the following
```
ports:
  - "8000:80"
```
to 

```
ports:
  - "80:80"
```

Make sure no other servers such as apache or nginx are listening on the same port before macking this change. 

To redeploy the image, run `docker compose up -d` in the directory where the docker-compose.yml file is located. 

### Recommendations
1. Prevent password login via shell (See sysadmin_test_observations_and_actions.md)

### Problems fixed
1. bash path variable and missing lines from code completion script
2. apt sources
3. docker missing and possible outdated packages
4. docker-compose file
