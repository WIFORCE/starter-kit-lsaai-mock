# WIFORCE Data Support Centre setup

The [original repos](https://github.com/GenomicDataInfrastructure/starter-kit-lsaai-mock) was forked and modified for the purpose of the DSC.

This repos is still just a mock and not meant for production, just to ease the deployment of the REMS-SDA-LSAAI stack for testing and dev purposes.

## Change Log

- 2026-08-17: changed the settings to use network_mode: "host". localhost/host.docker.internal was not resolving
  to the actual host as docker is set up with a different IP range to avoid colliding with the NFS (172.18.xxx.yyy)
- 2026-08-14: as we are using localhost not to have to open ports on the host, we created a `.env` file,
  modified the docker-compose.yaml to define extra_hosts and create the network as external so it can be
  joined by other compose stacks. We also created two configuration files for REMS and SDA.

## Setup

1. clone this repos
<!--
2. on the host, edit the `/etc/hosts` file so it resolves `host.docker.internal` as `localhost`
   ```bash
   sudo sh -c 'echo "127.0.0.1 host.docker.internal" >> /etc/hosts'
   ```
-->
2. create the docker network
   ```bash
   docker network create my_app_network
   ```
1. deploy
   ```bash
   docker compose up -d
   ```
1. test

  First run in the terminal
   ```bash
   curl -s http://pando.upsc.se:8080/oidc/.well-known/openid-configuration | jq .issuer
   ```
  should return `"http://pando.upsc.se:8080/oidc/"`
   
  Then [login into the front-end](http://pando.upsc.se:8080/oidc/authorize?response_type=code&client_id=rems-client&redirect_uri=http://pando.upsc.se:3000/oidc-callback&scope=openid%20profile%20email%20ga4gh_passport_v1)
  
  Click "Consent" and lookup the code in the returned URL (it will throw an error). It will look like: `http://pando.upsc.se:3000/oidc-callback?code=SOME_CODE`

  In the terminal, set the code as a variable
  ```bash
  export CODE=<CODE_FROM_ABOVE>
  ```

  Finally, test in the terminal
  ```bash
  curl -s -X POST http://localhost:8080/oidc/token \
  -u "rems-client:rems-secret" \
  -d "grant_type=authorization_code&code=${CODE}&redirect_uri=http://pando.upsc.se:3000/oidc-callback" \
  | jq . 
  ```

## TODO

1. Instead on relying on ~~localhost and the host.docker.internal mapping~~ `network_mode: "host"`, we could instead define the networks with static IPs, see that [Stack Overflow example](https://stackoverflow.com/questions/39493490/provide-static-ip-to-docker-containers-via-docker-compose)

1. Instead on relying on localhost and the host.docker.internal mapping, we could instead define the networks with static IPs, see that [Stack Overflow example](https://stackoverflow.com/questions/39493490/provide-static-ip-to-docker-containers-via-docker-compose)
