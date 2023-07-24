# confluence



https://hub.docker.com/r/atlassian/confluence-server



```
docker run -v /data/your-confluence-home:/var/atlassian/application-data/confluence --name="confluence" -d -p 8090:8090 -p 8091:8091 atlassian/confluence


docker run -v /home/keboom/confluence:/var/atlassian/application-data/confluence --name="confluence" -d -p 8090:8090 -p 8091:8091 atlassian/confluence


docker run --name postgresql -e POSTGRES_USER=myusername -e POSTGRES_PASSWORD=mypassword -p 5432:5432 -v /data:/var/lib/postgresql/data -d postgres

docker run --name postgresql -e POSTGRES_USER=keboom -e POSTGRES_PASSWORD=keboom -p 5432:5432 -v /home/keboom/postgresql:/var/lib/postgresql/data -d postgres

```





BQE3-2TKV-CN0H-PXOR



```
/opt/atlassian/confluence/confluence/WEB-INF/lib/com.atlassian.extras_atlassian-extras-decoder-v2-3.4.6.jar
```

```
docker cp confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/com.atlassian.extras_atl
assian-extras-decoder-v2-3.4.6.jar /tmp/atlassian-extras-2.4.jar
```

```
docker cp /tmp/com.atlassian.extras_atlassian-extras-decoder-v2-3.4.6.jar confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/com.atlassian.extras_atl
assian-extras-decoder-v2-3.4.6.jar 
```





```
java -jar atlassian-agent.jar -d -m keboom007@outlook.com -n BAT -p conf -o http://localhost:8090 -s BD8O-NYI2-9QYS-5SCH
```











https://www.cuiwei.net/p/1258785919   -------------这个可用！！！！



```
docker run --name postgresql -e POSTGRES_USER=keboom -e POSTGRES_PASSWORD=keboom -p 5432:5432 -v /home/keboom/postgresql:/var/lib/
postgresql/data -d postgres
```