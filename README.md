With this file you're about to run a Grafana with `docker-compose including grafana + prometheus + alertmanager + nodeexporter + cadvisor`.

First of all clone this repo:
```
git clone https://github.com/SaeedFazlollahzadeh/grafana-docker-compose-nginx.git
cd grafana-docker-compose-nginx
```

If you have pointed your DNS records of your domain to your current server, do steps **1)a** and **2)**, otherwise do only *1)b**.

# 1) Running and upping docker-compose #

1) **a**: If you do not have any pointed domains, open `docker-compose.yml` file with your preferred text editor and change all `127.0.0.1` to `0.0.0.0`. Mine is `vim docker-compose.yml`.

You also can run this to replace all occurences:
```
sed -e 's|127.0.0.1|0.0.0.0|g' -i docker-compose.yml
```
Then run `docker-compose up -d`.
Now your containers are up. Confirm by `docker ps -a`.

You can now confirm grafana works fine by either running `your_server_ip:3000` in a web browser or run the following command:
```
curl -IL your_server_ip:3000
```
You should see 200 HTTP Code.

1) **b**: If you have pointed your domains:

Then run `docker-compose up -d`.
Now your containers are up. Confirm by `docker ps -a`.

# 2) Installing nginx (Optional) #

Based on whether you're in a Debian-based or RHEL-based environment, run one of the following commands:
```
apt update && apt install -y nginx
```
for Debian-based.
```
yum install -y nginx
```
for RHEL-based.

Then run the following command to make sure `nginx` is started and will start during the boot:
```
systemctl start nginx && systemctl enable nginx
```
Now let us replace the `yourdomain.com` with your `actual-domain.com` with the following command:
```
sed -e 's|yourdomain.com|YOUR-ACTUAL-DOMAIN.COM|g' -i http-sites.conf
```
To have your domain start working now let's copy the config file of `nginx`:
```
cp -a http-sites.conf /etc/nginx/sites-available/http-sites.conf
ln -sf /etc/nginx/sites-available/http-sites.conf /etc/nginx/sites-enabled/http-sites.conf
systemctl reload nginx
```
If you have no errors, it means `nginx` is now reloaded fine.

You can confirm it by the following command:
```
curl -IL grafana.yourdomain.com
```
# 3) Installing certbot (Optional) #

If you want to use SSL for your grafana website, based on whether you're in a Debian-based or RHEL-based environment, run one of the following commands:
```
apt update && apt install -y certbot
```
for Debian-based.
```
yum install -y certbot
```
for RHEL-based.

Now we should run these commands to have a directory for `certbot` to create files and validate them:
```
mkdir -pv /home/ssl/acme-challenge
ln -sf /home/ssl/ /home/grafana-docker-compose-nginx/.well-known
```

There's an `ssl` file which runs `certbot` in `dry-run` or testing mode to avoid any limits for production environment.

Let us replace `domain.com` with our actual domain again:
```
sed -e 's|yourdomain.com|YOUR-ACTUAL-DOMAIN.COM|g' -i ssl
```
Now let us run this file:
```
bash ssl
```
If everything's OK, you should see `The dry run was successful` in your monitor., otherwise you should either wait for DNS Propagation or see what's wrong with it.

If everything's OK, now let's dry-run mode in `ssl` file:
```
sed -e 's|--dry-run||g' -i ssl
```
And now let's run it finally to have SSL certificates:
```
bash ssl
```
Now that both `nginx` and SSL certificates are correct and there are no errors, you can copy `https-sites.conf` to `nginx` directory to finish it.

Now let us replace the `yourdomain.com` with your `actual-domain.com` with the following command:
```
sed -e 's|yourdomain.com|YOUR-ACTUAL-DOMAIN.COM|g' -i https-sites.conf
```
As we will surely error because of having duplicate names, let us remove `http-sites.conf` in `nginx`:
```
rm -fi /etc/nginx/sites-enabled/http-sites.conf
```
To have your domain start working now let's copy the config file of `nginx`:
```
cp -a http-sites.conf /etc/nginx/sites-available/http-sites.conf
ln -sf /etc/nginx/sites-available/http-sites.conf /etc/nginx/sites-enabled/http-sites.conf
systemctl reload nginx
```
If you have no errors, it means `nginx` is now reloaded fine.

You can confirm it by the following command:
```
curl -IL grafana.yourdomain.com
```
You should see something like the following output:
```
root@server:~## curl -IL grafana.yourdomain.com
HTTP/1.1 301 Moved Permanently
Server: nginx/1.18.0 (Ubuntu)
Date: Sat, 25 Feb 2023 21:50:20 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: https://grafana.yourdomain.com/

HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Sat, 25 Feb 2023 21:50:20 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Cache-Control: no-cache
Expires: -1
Pragma: no-cache
X-Content-Type-Options: nosniff
X-Xss-Protection: 1; mode=block
```
