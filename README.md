
# Projet DevSecOps – Déploiement Multi-Applications avec Docker, Nginx et Reverse Proxy

## Auteur : Younes KOUBOUSSE / E5 API  
Date : 11 juillet 2025  
École : ESTIAM Paris  
Module : E5 - DevSecOps  

---

##  Contexte du projet

Ce projet s’inscrit dans le cadre du module DevSecOps de l’école ESTIAM. Il a pour objectif de nous initier à la conteneurisation d’applications, à la gestion des dépendances, au déploiement multi-services via Docker Compose, ainsi qu’à l’intégration d’un reverse proxy (Nginx). Le tout dans une logique d'automatisation et d'orchestration d’environnements logiciels.

---

##  Objectifs pédagogiques

- Créer un environnement isolé pour chaque application via des conteneurs Docker.
- Utiliser `Dockerfile` pour construire les images.
- Mettre en place un `docker-compose.yml` pour orchestrer l’ensemble.
- Utiliser un reverse proxy (Nginx) pour accéder aux applications via des chemins URL dédiés.
- Fournir une documentation complète pour décrire l’infrastructure mise en place.

---

##  Étape 1 – Structure du projet

La première étape a été d’identifier les projets à déployer. J’ai choisi 4 applications web différentes pour illustrer plusieurs technologies :

1. **flask-soft** : une application simple en Flask
2. **flask-htmlx** : une application Flask avec HTMLx (HTMX)
3. **django-soft** : une application Django
4. **rocket-ecommerce** : une application Node.js de e-commerce

Chaque application a été décompressée dans un dossier distinct. Ensuite, j’ai créé les fichiers suivants à la racine du projet :

- `docker-compose.yml` → pour l’orchestration
- `nginx.conf` → pour le reverse proxy
- `README.md` → pour la documentation

Structure finale :
```
projet-devsecops/
 ┣ flask-soft/
 ┣ flask-htmlx/
 ┣ django-soft/
 ┣ rocket-ecommerce/
 ┣ docker-compose.yml
 ┣ nginx.conf
 ┗ README.md
```

---

##  Étape 2 – Dockerisation des applications

J’ai ajouté un fichier `Dockerfile` dans chaque dossier :

###  Exemple pour Flask (flask-soft) :
```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
```

###  Exemple pour Django :
```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

###  Exemple pour Node.js (Rocket) :
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "run", "dev"]
```

Chaque `Dockerfile` est spécifique à la technologie de l’application.

---

##  Étape 3 – Orchestration avec Docker Compose

Le fichier `docker-compose.yml` centralise la configuration des 6 services :
- Les 4 applications
- Le reverse proxy (nginx)
- La passerelle Stripe simulée (mock)

```yaml
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
```

---

##  Étape 4 – Reverse Proxy avec Nginx

Le fichier `nginx.conf` permet de rediriger les URLs vers les bons conteneurs.

```nginx
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
```

Cela permet d'accéder aux apps via :
- `http://localhost/flask-htmlx/`
- `http://localhost/django/`
- `http://localhost/rocket/`

---

##  Étape 5 – Lancement de la stack

Dans un terminal :
```bash
docker-compose build         # Construction des images
docker-compose up -d         # Démarrage des conteneurs
docker ps                    # Vérification des services actifs
```

---

##  Étape 6 – Test et vérification

J’ai vérifié chaque URL dans le navigateur. Voici les résultats :
- ✅ `http://localhost:5001` → Flask Soft accessible directement
- ✅ `http://localhost/flask-htmlx/`
- ✅ `http://localhost/django/`
- ✅ `http://localhost/rocket/`

---

##  Étape 7 – Stripe CLI

J’ai intégré un service Stripe fictif avec `stripe/stripe-cli` pour simuler les paiements.

---

##  Étape 8 – Gestion des erreurs et relances

Pour relancer après un changement :
```bash
docker-compose down
docker-compose up --build -d
```

Pour lire les logs :
```bash
docker-compose logs -f
```

---

##  Étape 9 – Dépôt GitHub

1. Création du repo sur GitHub
2. Initialisation locale :
```bash
git init
git remote add origin https://github.com/MONUTILISATEUR/projet-devsecops.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

---

##  Étape 10 – Répartition de l’équipe (exemple)

| Membre          | Rôle                         |
|----------------|------------------------------|
| Younes KOUBOUSSE    | Chef de projet / Docker      |
| Mohammed BOUHACHLAF | Déploiement / Nginx / Reverse proxy |
| Reda ZITOUNI  | Dockerfiles / CI / Debug     |
| Ayoub MAD  | Testing / Monitoring     |
| Ziad Fourati  | Monitoring / Debug    |

---

##  Schéma d’architecture

```
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
```

---

## Résultat final

-  4 applications dockerisées
-  Reverse proxy fonctionnel
-  Documentation complète
-  Lancement automatisé avec Docker Compose

---

© 2025 – Younes KOUBOUSSE – ESTIAM Paris
