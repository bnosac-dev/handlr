# handlr

`handlr` makes R functions available as API endpoints. It is the less opinionated cousin of [OpenCPU](https://github.com/opencpu/opencpu).



#### Shared Features with OpenCPU

* Built on Apache2 and rApache
* Each request handled in seperate R fork 
* Borrows code e.g. for parsing http requests



#### Distinguishing Features

* Doesn't expose your R code through GET requests
* You decide what R functions can be run
* Easy user management with [`authr`](https://github.com/alexvpickering/authr)


# Getting Started

Assumes Ubuntu 16.04. See the [wiki](https://github.com/alexvpickering/handlr/wiki) for a more thorough intro and `authr` integration.

Install Apache2, rApache, and add an example site:

```
sudo apt update
sudo apt install apache2

sudo add-apt-repository ppa:opencpu/rapache
sudo apt-get update
sudo apt-get install libapache2-mod-r-base

sudo touch /etc/apache2/sites-available/example.conf
sudo vi /etc/apache2/sites-available/example.conf
```

Type `i` to insert then paste the following:

```apache
LoadModule R_module /usr/lib/apache2/modules/mod_R.so

<Location /api>
	SetHandler r-handler
	RFileHandler /var/www/R/entry.R
</Location>
```

Save (type `:wq` then hit enter). To enable the above site and create the `entry.R` that rApache will run:


```
sudo a2ensite example
sudo service apache2 reload

sudo mkdir /var/www/R
sudo touch /var/www/R/entry.R
sudo vi /var/www/R/entry.R
```

Type `i` to insert then paste the following:

```R
setHeader(header = "X-Powered-By", value = "rApache")

# functions exported by 'packages' can be used as endpoints
packages <- c('your_package')

handlr::handle(SERVER, GET, packages)
DONE
```

Save and exit as before. Install `handlr` and `your_package`, making sure they will be available to the `www-data` user that Apache2 runs under:

```R
# system dependencies
sudo apt install libssl-dev 
sudo apt install libcurl4-openssl-dev
sudo apt install libsodium-dev
sudo apt install libsasl2-dev

sudo R
.libPaths('/usr/local/lib/R/site-library')

install.packages('sys', configure.vars = 'NO_APPARMOR=1')
install.packages('devtools')

devtools::install_github(c('alexvpickering/handlr', 'your_github_username/your_package'))

```
