I spent way too much time to get a local setup for fleet running with docker-compose. The process is poorly documented and I decided to fix it since I have a hunch I will need to do this some more and it's' a very non trivial process currently. This is valid as of 8.5.2; this may change of course.

Fleet requires security to be active in Elastic and setting this up requires a few manual steps that below

## Step 1: start elasticsearch

```bash
docker-compose up elasticsearch -d
```

wait for this to start and verify it comes up:

```bash
curl http://elastic:secret@localhost:9200
```

It should respond. Note the user/password in here. This is the default user and the password configured in docker-compose.

## Step 2: Create Service account for kibana

```bash
curl -X POST http://elastic:secret@localhost:9200/_security/service/elastic/kibana/credential/token/kibana-token
```

open `kibana.config.yml` and add the token that you just created and set it to the `elasticsearch.serviceAccountToken` property. Paste the `value` value of the returned json object when you created the service account.

## Step 3: Verify you can login to Kibana

Login as `elastic` with password `secret` (or whatever you changed in the `docker-compose.yml` file)

## Step 4: Verify the service account persists

Shut down everything. This also deletes ephemeral volumes.

```bash
docker-compose down
```

And bring it back up everything

```bash
docker-compose up -d elasticsearch kibana
```

## Step 5: Configuer fleet-server

Create another service account for fleet:

```bash
curl -X POST http://elastic:secret@localhost:9200/_security/service/elastic/fleet-server/credential/token/fleet-token
```

Take the token and use it in the docker-compose file to set the `FLEET_SERVER_SERVICE_TOKEN` for the fleet-server.

Start `fleet-server`

```bash
docker-compose up fleet-server -d
```

Verify it starts without errors. Go to Kibana and open `Fleet` and under `Add a server` generate a policy for `https://localhost:8220`.


