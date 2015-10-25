# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

site_slug = File.basename( Dir.glob("wp-content/themes/*").reject { |e| e.include? ".php" }.first.gsub(/\.(com|org|net|co\.uk|co|edu|info|biz)/){|m| ".dev"} )

$script = <<SCRIPT
ping_result="$(ping -c 2 8.8.4.4 2>&1)"
if [[ $ping_result != *bytes?from* ]]; then
  ping_result="$(ping -c 2 4.2.2.2 2>&1)"
fi

if [[ $ping_result == *bytes?from* ]]; then
  echo "\nExternal network connection established, packages up to date."

  exists_mysql="$(service mysql status)"
  if [[ "mysql: unrecognized service" != "${exists_mysql}" ]]; then
    echo -e "\nProvisioning vagrant box for first time."

    apt-get update
    # apt-get upgrade -y
    apt-get install -y vim wget htop git python-software-properties software-properties-common

    add-apt-repository ppa:nginx/stable
    apt-get update
    apt-get install -y nginx

    apt-key adv --yes --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
    add-apt-repository 'deb http://ftp.utexas.edu/mariadb/repo/5.5/ubuntu trusty main'
    apt-get update
    sudo debconf-set-selections <<< 'mariadb-server-5.5 mysql-server/root_password password root'
    sudo debconf-set-selections <<< 'mariadb-server-5.5 mysql-server/root_password_again password root'
    apt-get install -y mariadb-server
    sed -i "s/bind-address/# bind-address/g" /etc/mysql/my.cnf
    mysql -uroot -proot -Bce "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION; FLUSH PRIVILEGES;"
    /etc/init.d/mysql restart
    # mysql_secure_installation

    apt-get install -y memcached

    wget -O - http://dl.hhvm.com/conf/hhvm.gpg.key | sudo apt-key add -
    echo deb http://dl.hhvm.com/ubuntu trusty main | sudo tee /etc/apt/sources.list.d/hhvm.list
    apt-get update
    apt-get install -y hhvm
    /usr/share/hhvm/install_fastcgi.sh
    /etc/init.d/hhvm restart
    sudo update-rc.d hhvm defaults

    # apt-get install -y sendmail
    # sendmailconfig

    echo postfix postfix/main_mailer_type select Internet Site | debconf-set-selections
    echo postfix postfix/mailname string vvv | debconf-set-selections
    apt-get install -y postfix

    sed -i "s/# gzip_types/gzip_types/g" /etc/nginx/nginx.conf
    sed -i "s/worker_processes 2;/worker_processes 4;/g" /etc/nginx/nginx.conf

    cat << EOF > /etc/nginx/hhvm.conf
location ~ \\.(hh|php)\\$ {
    fastcgi_keep_conn on;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME \\$document_root\\$fastcgi_script_name;
    include        fastcgi_params;
}
EOF

    sed -i "/as directory/{n;d}" /etc/nginx/sites-available/default
    awk '/as directory/{print;print "\t\tdeny all;";next}1' /etc/nginx/sites-available/default > /etc/nginx/sites-available/default.tmp && mv /etc/nginx/sites-available/default.tmp /etc/nginx/sites-available/default
    sed -i "s/as directory/already patched/g" /etc/nginx/sites-available/default

  else
    echo -e "\nVagrant box already provisioned."
  fi
else
  echo "\nNo external network available. Package installation and maintenance skipped."
fi

cat << EOF > /etc/nginx/sites-available/#{site_slug}
server {
  listen 80;
  server_name www.#{site_slug};
  rewrite ^ http://#{site_slug}/\\$1 permanent;
}

server {
  listen *:80;
  server_name #{site_slug};
  location ~ /(\\.|wp-config.php|Vagrantfile) {
    deny all;
  }
  location / {
    root /var/www/#{site_slug}/public;
    index index.php index.html index.htm;
    try_files \\$uri \\$uri/ /index.php?q=\\$uri&\\$args;
  }
  location ~ \\.(hh|php)\\$ {
    root /var/www/#{site_slug}/public;
    fastcgi_keep_conn on;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME \\$document_root\\$fastcgi_script_name;
    include        fastcgi_params;
  }
}
EOF
ln -sf /etc/nginx/sites-available/#{site_slug} /etc/nginx/sites-enabled/#{site_slug}
mkdir -p /var/www/#{site_slug}
chown -R vagrant:vagrant /var/www/#{site_slug}
ln -sf /vagrant /var/www/#{site_slug}/public
service nginx restart
mysql -uroot -proot -Bce "create database #{site_slug.gsub("."){ |m| "_" }};"
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network :private_network, ip: "10.0.5.4"

  config.vm.hostname = "vvv"

  config.ssh.forward_agent = true

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder ".", "/vagrant", type: "nfs"

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "512"]
    vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
  end

  if defined? VagrantPlugins::HostsUpdater
    config.hostsupdater.aliases = [site_slug]
  end

  config.vm.provision :shell, :inline => $script
end
