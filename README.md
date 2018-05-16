# How to run Symfony on "Vagrant"
This is a misleading title by design. The idea is to run
[Symfony](https://symfony.com/) using [Vagrant](https://www.vagrantup.com/)
but behind the scenes it's running Docker instead of
[VirtualBox](https://www.virtualbox.org/) or
[VMWare](https://www.vmware.com/products/personal-desktop-virtualization.html).

# Why?
1. People get crazy when we change tooling. So keep same commands as
`vagrant up` and `vagrant halt` will help to improve without hurt feelings.
2. The file system integration to change one file and see the changes on
the running service is way better this way.
3. Because we can :p

# Recipe

## Ingredients
- Vagrant
- Docker
- Composer

## How to prepare
Prepare a Symfony project from skeleton using composer:
```bash
mkdir my-new-legacy
cd my-new-legacy
composer create-project symfony/website-skeleton .
composer require symfony/apache-pack
```

Create a `Dockerfile` to run Symfony:
```docker
FROM php:7.1-apache
ENV APACHE_DOCUMENT_ROOT /symfony/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
RUN sed -ri -e 's!index.php!app.php index.php!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
COPY . /symfony
WORKDIR /symfony
```

Finally add the `Vagrantfile` that will make the magic:
```ruby
Vagrant.configure("2") do |config|
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.synced_folder ".", "/symfony"
  config.vm.provider "docker" do |d|
    d.build_dir = "."
  end
end
```

That's it! You can now taste with you favorite Vagrant commands:
```bash
vagrant up
```

As bonus you can check the container logs with
```bash
vagrant docker-logs
```
or follow logs with
```bash
vagrant docker-logs -f
```
