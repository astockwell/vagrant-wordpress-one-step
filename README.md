WordPress "In One Step" Vagrantfile
===================================

Complete WordPress development environment with one `vagrant up`


## Usage

1. Download a fresh WordPress installation to the same root folder as the Vagrantfile.
2. Delete all unused themes/folders from `wp-content/themes` and install desired theme folder.
3. Remane the desired theme folder to the project's URL, e.g. rename the `twentyfifteen` folder to `example.com` if the website will ultimately be hosted at example.com. The name of this folder sets the convention that everything else is based on.
4. Run `vagrant up`.
5. Using the convention of the theme folder name (and expecting it to be structured like as a domain name), the Vagrantfile will create an Nginx profile at `example.dev` for local development, and will create a database on the Vagrant guest named `example_dev` for development purposes.
6. Once `vagrant up` completes, you should be able to go immediately to `example.dev` (or whatever your theme folder's dev URL would be following the conventions laid out above) and start working!

** Be sure to back up any MySQL database(s) from the Vagrant guest box before destroying it!!! **


## Contents

- Ubuntu 14.04 x64
- Nginx
- HHVM (HipHop Virtual Machine)
- MariaDB 5.5


## Requirements

- [Vagrant](https://www.vagrantup.com/)
- [Vagrant::Hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater)
