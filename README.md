Projet DevSecOps – Déploiement Multi-Applications avec Docker, Nginx et Reverse Proxy
Auteur : Ayoub MAD / E5 WMD
Date : 11 juillet 2025
École : ESTIAM Paris
Module : E5 – WMD

Contexte du projet
Ce projet a été réalisé dans le cadre du module DevSecOps à ESTIAM. Il avait pour objectif d’apprendre à conteneuriser différentes applications web, à gérer leurs dépendances et à orchestrer leur déploiement grâce à Docker Compose. L’intégration d’un reverse proxy avec Nginx permet de simplifier l’accès aux services et de structurer l’architecture. L’ensemble vise à favoriser l’automatisation et la reproductibilité, conformément aux bonnes pratiques DevSecOps.

Objectifs pédagogiques
Créer des conteneurs isolés pour chaque application.

Construire les images à l’aide de Dockerfile adaptés.

Orchestrer l’ensemble des services avec un fichier docker-compose.yml.

Configurer un reverse proxy Nginx pour centraliser les accès via des URL dédiées.

Rédiger une documentation claire et complète décrivant l’infrastructure.

Étape 1 – Organisation et structure du projet
La première étape a consisté à sélectionner quatre applications web, chacune reposant sur une technologie différente, afin d’illustrer la diversité des environnements :

flask-soft : application simple développée avec Flask.

flask-htmlx : application Flask intégrant HTMX.

django-soft : application Django.

rocket-ecommerce : application Node.js orientée e-commerce.

Chaque projet a été placé dans un dossier séparé. À la racine, plusieurs fichiers ont été créés :

docker-compose.yml pour coordonner tous les services.

nginx.conf pour la configuration du reverse proxy.

README.md pour documenter le déploiement et l’architecture.

Structure finale :

Copier le code
projet-devsecops/
 ┣ flask-soft/
 ┣ flask-htmlx/
 ┣ django-soft/
 ┣ rocket-ecommerce/
 ┣ docker-compose.yml
 ┣ nginx.conf
 ┗ README.md
Étape 2 – Création des Dockerfiles
Chaque application possède un Dockerfile adapté :

Exemple Flask :

dockerfile
Copier le code
FROM python:3.10-slim
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
Exemple Django :

dockerfile
Copier le code
FROM python:3.10-slim
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
Exemple Node.js :

dockerfile
Copier le code
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "run", "dev"]
Étape 3 – Orchestration avec Docker Compose
Le fichier docker-compose.yml centralise la configuration de l’ensemble des services : les quatre applications, le reverse proxy Nginx et une passerelle Stripe simulée.

yaml
Copier le code
services:
  flask-soft:
    build:
      context: ./flask-soft
    ports:
      - "5001:5000"

  flask-htmlx:
    build:
      context: ./flask-htmlx
    expose:
      - "5000"

  django-soft:
    build:
      context: ./django-soft
    expose:
      - "8000"

  rocket:
    build:
      context: ./rocket-ecommerce
    expose:
      - "3000"

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"

  stripe:
    image: stripe/stripe-cli
    command: listen --forward-to http://host.docker.internal:5001/webhook
Étape 4 – Configuration du reverse proxy Nginx
Le fichier nginx.conf redirige les requêtes vers les bons conteneurs selon le chemin URL :

nginx
Copier le code
server {
    listen 80;

    location /flask-htmlx/ {
        proxy_pass http://flask-htmlx:5000;
    }

    location /django/ {
        proxy_pass http://django-soft:8000;
    }

    location /rocket/ {
        proxy_pass http://rocket:3000;
    }
}
Accès via :

http://localhost/flask-htmlx/

http://localhost/django/

http://localhost/rocket/

Étape 5 – Démarrage de la stack
Les commandes principales pour construire et lancer l’environnement :

bash
Copier le code
docker-compose build         # Construire les images
docker-compose up -d         # Démarrer les services
docker ps                    # Vérifier les conteneurs en cours d’exécution
Étape 6 – Vérification et tests
Les différents points d’accès ont été vérifiés :

✅ http://localhost:5001 (Flask Soft en accès direct)

✅ http://localhost/flask-htmlx/

✅ http://localhost/django/

✅ http://localhost/rocket/

Étape 7 – Simulation des paiements avec Stripe CLI
Un service stripe/stripe-cli a été intégré pour simuler la réception de paiements et tester les webhooks.

Étape 8 – Maintenance et relance des services
Pour relancer après des modifications :

bash
Copier le code
docker-compose down
docker-compose up --build -d
Consulter les logs :

bash
Copier le code
docker-compose logs -f
Étape 9 – Publication sur GitHub
Mise en ligne du projet :

bash
Copier le code
git init
git remote add origin https://github.com/MONUTILISATEUR/projet-devsecops.git
git add .
git commit -m "Initial commit"
git push -u origin main
Étape 10 – Répartition des rôles au sein de l’équipe
Membre	Rôle
Ayoub MAD	Testing / Monitoring
Younes KOUBOUSSE	Chef de projet / Docker
Mohammed BOUHACHLAF	Déploiement / Nginx / Reverse proxy
Reda ZITOUNI	Dockerfiles / CI / Debug
Ziad Fourati	Monitoring / Debug

Schéma d’architecture
swift
Copier le code
         [ Utilisateur ]
               |
               v
         [ Nginx Proxy ]
        /       |       \
 /flask-htmlx/ /django/  /rocket/
     |           |          |
flask-htmlx  django-soft  rocket
               |
         [ Stripe CLI ]
Résultat final
Quatre applications conteneurisées

Reverse proxy opérationnel

Documentation complète

Déploiement et maintenance automatisés avec Docker Compose

© 2025 – Ayoub MAD – ESTIAM Paris