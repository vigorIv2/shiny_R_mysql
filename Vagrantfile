
# customization section, tweak these parameters if you know what you are doing
$mysql_root_password='xyzzy'
$address_subnet='192.168.222'
$mysql_address="#{$address_subnet}.10"
$mysql_host='vm-mysql-shiny'
$shiny_host='vm-web-shiny'
$shiny_pass='5hinyPa55'

$generate_my_cnf = <<SCRIPT0
echo \'[client]\' >> .my.cnf 
echo \'user = shiny\' >> .my.cnf
echo \'password = #{$shiny_pass}\' >> .my.cnf
echo \'host = #{$mysql_address}\' >> .my.cnf
SCRIPT0

$shiny_R_script = <<SCRIPT2
#!/bin/bash

#{$generate_my_cnf}

sudo apt-get install curl -y
sudo apt-get update
sudo apt-get -q -y --force-yes install pandoc texlive-latex-base texlive-latex-extra texlive-latex-recommended texlive-fonts-recommended texlive-xetex r-base libcurl4-gnutls-dev r-base-dev r-cran-rmysql gdebi-core libmysqlclient-dev mysql-client-core-5.6
sudo apt-get -q -y --force-yes autoremove
echo "R has been installed"

sudo su - -c "R -e \\"install.packages\('shiny', repos='http://cran.rstudio.com/'\)\\""
sudo su - -c "R -e \\"install.packages\('RCurl', repos='http://cran.rstudio.com/'\)\\""
sudo su - -c "R -e \\"install.packages\('devtools', repos='http://cran.rstudio.com/'\)\\""
sudo su - -c "R -e \\"devtools::install_github\('hadley/devtools'\)\\""
echo "R + shiny has been installed"

wget http://download3.rstudio.org/ubuntu-12.04/x86_64/shiny-server-1.3.0.403-amd64.deb
sudo gdebi -n shiny-server-1.3.0.403-amd64.deb

#sudo su - -c "chown -R shiny /usr/local/lib/R" 
#sudo su - -c "chgrp -R shiny /usr/local/lib/R" 
#sudo su - -c "R -e \\"chooseCRANmirror(86); install.packages\('rmarkdown'\)\\""
#sudo su - -c "R -e \\"devtools::install_github\('rstudio/rmarkdown'\)\\""
sudo su - -c "R -e \\"install.packages\('rmarkdown', dependencies = TRUE\)\\""

SCRIPT2

#install.packages('knitr', dependencies = TRUE)
#install.packages('yaml', dependencies = TRUE)
#install.packages('caTools', dependencies = TRUE)
#install.packages('rmarkdown', dependencies = TRUE)

#install.packages('knitr', repos = c('http://rforge.net', 'http://cran.rstudio.org'), type = 'source')
#install.packages('yaml', repos = c('http://rforge.net', 'http://cran.rstudio.org'), type = 'source')
#install.packages('caTools', repos = c('http://rforge.net', 'http://cran.rstudio.org'), type = 'source')

$mysql_server_script = <<SCRIPT3
#!/bin/bash

#{$generate_my_cnf}

echo mysql-server-5.6 mysql-server/root_password password #{$mysql_root_password} | sudo debconf-set-selections
echo mysql-server-5.6 mysql-server/root_password_again password #{$mysql_root_password} | sudo debconf-set-selections

sudo apt-get install curl -y
sudo apt-get update
sudo apt-get -y --force-yes install mysql-server-5.6
sudo apt-get -y --force-yes autoremove
sudo sed -i "s/bind-address.*/bind-address = #{$mysql_address}/" /etc/mysql/my.cnf
sudo service mysql restart
mysql -B -h#{$mysql_address} -uroot -p#{$mysql_root_password} -e "create database shiny;" > /dev/null
mysql -B -h#{$mysql_address} -uroot -p#{$mysql_root_password} -e "GRANT ALL ON shiny.* TO 'shiny'@'%' IDENTIFIED BY '#{$shiny_pass}';" > /dev/null
mysql -B -h#{$mysql_address} -uroot -p#{$mysql_root_password} -e "flush privileges;" > /dev/null

SCRIPT3

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "trusty64"
  config.vm.box_url = "https://github.com/kraksoft/vagrant-box-ubuntu/releases/download/14.04/ubuntu-14.04-amd64.box"

  # Manage /etc/hosts on host and VMs
  config.hostmanager.enabled = false
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.hostmanager.ignore_private_ip = false

  config.vm.define :mysql_shiny do |mysql_shiny|
    mysql_shiny.vm.provider :virtualbox do |v|
      v.name = "vm-mysql-shiny"
      v.customize ["modifyvm", :id, "--groups", "/Shiny", "--memory", "1024" ]
    end
    mysql_shiny.vm.hostname = $mysql_host
    mysql_shiny.vm.provision :hostmanager
    mysql_shiny.vm.network "forwarded_port", guest: 3306, host:3306 
    mysql_shiny.vm.provision :shell, :inline => $mysql_server_script
    mysql_shiny.vm.network :private_network, ip: "#{$mysql_address}" 
  end

  config.vm.define :web_shiny do |web_shiny|
    web_shiny.vm.provider :virtualbox do |v|
      v.name = "vm-web-shiny"
      v.customize ["modifyvm", :id, "--groups", "/Shiny", "--memory", "2048" ]
    end
    web_shiny.vm.provision :shell, :inline => $shiny_R_script
    web_shiny.vm.hostname = $shiny_host
    web_shiny.vm.provision :hostmanager
    web_shiny.vm.network "forwarded_port", guest: 3838, host: 3838
    web_shiny.vm.network :private_network, ip: "#{$address_subnet}.11" 
  end

end
