# Docker Compose CMS

This project is designed to replace shared hosting providers as well as the monolithic CMS architecture. In this project we utilize [Strapi](https://strapi.io/) CMS as a headless CMS, with the intension of creating a React application (Or other framework) and hosting on a platform like Vercel, Netlify or GithubPages.

## Start

``` bash
docker-compose -f docker-compose.dev.yml up -d --build
docker-compose -f docker-compose.dev.yml down

rm -rf ./docker/volumes
```

## Refresh Nginx

``` bash
docker exec docker-compose-cms-nginx-1 nginx -s reload
```

## Obtain SSL Certificate

``` bash
# Add --dry-run flag to the end of the command to test certbot
docker exec -i docker-compose-cms-maintenance-1 certbot certonly --webroot --webroot-path /var/certbot/ -d example.com
```

### Configure MailServer

You will first need to add the following DNS Records to your nameserver.

| Type | Name |     Content     |
|:----:|:----:|:---------------:|
|   A  | @    | ServerIP        |
|   A  | mail | ServerIP        |
|  MX  | @    | mail.\<domain>  |
|  TXT | @    | v=spf1 mx ~all  |

``` bash
# Generate SSL certificate for mail domain Via certbot. Add ```--dry-run``` to the end of the command to test first.
docker exec -i docker-compose-deploy_base_1 certbot certonly --webroot --webroot-path /var/certbot/ -d mail.example.com

# Create postmaster account, other accounts can be added in the same way.
docker exec -ti docker-compose-cms-ailserver-1 setup email add user@example.com <password>

# Generate DKIM key and record
docker exec -ti docker-compose-cms-ailserver-1 setup config dkim

# Display generated DKIM record.
cat ./docker/volumes/mailserver/config/opendkim/keys/<domain>/mail.txt
```

Finally you will need to add one more record to your nameserver with the output of the final command in the following format:

| Type |       Name      |                       Content                      |
|:----:|:---------------:|:--------------------------------------------------:|
|  TXT | mail._domainkey | v=\<value>;  h=\<value>;  k=\<value>;  p=\<value>; |

You can now use an email client to connect to your mail server with the following:


#### Incomming IMAP

|    Key   |      Value        |
|:--------:|:-----------------:|
| Hostname | mail.\example.com |
| Port     | 143               |
| Security | STARTTLS          |


#### Outgoing

|    Key   |      Value        |
|:--------:|:-----------------:|
| Hostname | mail.\example.com |
| Port     | 587               |
| Security | STARTTLS          |
