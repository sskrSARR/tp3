# TP3 – Déploiement d’une application Full Stack avec Vagrant

Ce projet démontre le **déploiement d’une application full stack** composée de :

* **Frontend** : Angular servi par **Nginx**
* **Backend** : API **Spring Boot**
* **Base de données** : **MySQL**

L’infrastructure est simulée avec **3 machines virtuelles Vagrant** afin de reproduire une architecture distribuée proche d’un environnement réel.

---

# Architecture

```
Navigateur
    │
    ▼
http://localhost:8081
server-front
(Nginx + Angular)
192.168.56.20
    │
    ▼
server-back
(Spring Boot API)
192.168.56.19:8080
    │
    ▼
server-dba
(MySQL)
192.168.56.18
```

---

# Prérequis

Avant de lancer ce projet, assurez-vous d’avoir installé :

* Vagrant
* VirtualBox
* Git
* Node.js (pour tester Angular localement si besoin)

---

# Structure des machines virtuelles

| Machine      | Rôle                     | IP            |
| ------------ | ------------------------ | ------------- |
| server-dba   | Base de données MySQL    | 192.168.56.18 |
| server-back  | Backend Spring Boot      | 192.168.56.19 |
| server-front | Frontend Angular + Nginx | 192.168.56.20 |

---

# Vagrantfile

```ruby
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/jammy64"

  config.vm.define "server-dba" do |dba|
    dba.vm.hostname = "server-dba"
    dba.vm.network "private_network", ip: "192.168.56.18"
  end

  config.vm.define "server-back" do |back|
    back.vm.hostname = "server-back"
    back.vm.network "private_network", ip: "192.168.56.19"
    back.vm.network "forwarded_port", guest: 8080, host: 8080

    back.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  config.vm.define "server-front" do |front|
    front.vm.hostname = "server-front"
    front.vm.network "private_network", ip: "192.168.56.20"
    front.vm.network "forwarded_port", guest: 80, host: 8081

    front.vm.provider "virtualbox" do |vb|
      vb.memory = 4096
      vb.cpus = 2
    end
  end

end
```

---

# Lancement des machines virtuelles

Dans le dossier du projet :

```bash
vagrant up
```

Connexion à une machine :

```bash
vagrant ssh server-back
```

---

# Déploiement de la base de données

Connexion :

```bash
vagrant ssh server-dba
```

Installation de MySQL :

```bash
sudo apt update
sudo apt install mysql-server -y
```

Créer la base de données :

```sql
CREATE DATABASE productdb;
```

Créer un utilisateur :

```sql
CREATE USER 'appuser'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON productdb.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
```

Autoriser les connexions externes en modifiant :

```
/etc/mysql/mysql.conf.d/mysqld.cnf
```

Remplacer :

```
bind-address = 0.0.0.0
```

Puis redémarrer MySQL :

```bash
sudo systemctl restart mysql
```

---

# Déploiement du Backend (Spring Boot)

Connexion :

```bash
vagrant ssh server-back
```

Installer Java et Maven :

```bash
sudo apt update
sudo apt install openjdk-17-jdk maven git -y
```

Cloner le backend :

```bash
git clone https://github.com/sskrSARR/FirstJavaSpring-Product.git
cd FirstJavaSpring-Product
```

Configurer la connexion MySQL :

```
src/main/resources/application.properties
```

```
spring.datasource.url=jdbc:mysql://192.168.56.18:3306/productdb
spring.datasource.username=appuser
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
server.address=0.0.0.0
```

Lancer l’application :

```bash
mvn spring-boot:run
```

Test API :

```
http://localhost:8080/products
```

---

# Déploiement du Frontend (Angular)

Connexion :

```bash
vagrant ssh server-front
```

Installer Node et Nginx :

```bash
sudo apt update
sudo apt install nginx git curl -y
```

Installer Node.js :

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install nodejs -y
```

Cloner le frontend :

```bash
git clone https://github.com/sskrSARR/front_products.git
cd front_products
npm install
```

Modifier l’URL du backend :

```
src/environments/environment.ts
```

```
apiUrl: 'http://192.168.56.19:8080/products'
```

Build Angular :

```bash
ng build --configuration production
```

Copier le build dans Nginx :

```bash
sudo rm -rf /var/www/html/*
sudo cp -r dist/angularforjava/browser/* /var/www/html/
```

Redémarrer Nginx :

```bash
sudo systemctl restart nginx
```

---

# Accès à l’application

Frontend :

```
http://localhost:8081
```

Backend API :

```
http://localhost:8080/products
```

---

# Captures

<img width="1731" height="800" alt="image" src="https://github.com/user-attachments/assets/f0aec08e-ccb3-4692-b45e-097727994694" />
<img width="1731" height="800" alt="image" src="https://github.com/user-attachments/assets/e7d99d17-bd88-4026-b32c-85faef4767a1" />
<img width="1731" height="800" alt="image" src="https://github.com/user-attachments/assets/89567dad-9d76-4a4f-8f01-37602771ed10" />
<img width="1731" height="800" alt="image" src="https://github.com/user-attachments/assets/45341f37-838f-42ea-ad72-4417ccafdb97" />
<img width="1919" height="993" alt="image" src="https://github.com/user-attachments/assets/38c31bdd-3c7f-4df5-be12-877bd397afd5" />






