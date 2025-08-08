# DATASCIENTEST JENKINS EXAM - Jet-XIII

Création d'un multibranch pipeline pour gérer les 4 environnements demandés:
dev qa staging prod(sur la branche master)

Pour un meilleur fonctionement du code, j'ai codé un deployment par service: 

.
├── FETCH_HEAD
├── Jenkinsfile
├── README.md
├── cast-service
│   ├── Dockerfile
│   ├── app
│   │   ├── api
│   │   │   ├── casts.py
│   │   │   ├── db.py
│   │   │   ├── db_manager.py
│   │   │   └── models.py
│   │   └── main.py
│   └── requirements.txt
├── docker-compose.yml
├── fastapiapp
│   ├── Chart.lock
│   ├── Chart.yaml
│   ├── charts
│   │   └── postgresql-12.1.5.tgz
│   ├── templates
│   │   ├── NOTES.txt
│   │   ├── _helpers.tpl
│   │   ├── deployment-cast.yaml
│   │   ├── deployment-movie.yaml
│   │   ├── hpa.yaml
│   │   ├── ingress.yaml
│   │   ├── service-cast.yaml
│   │   ├── service-movie.yaml
│   │   ├── serviceaccount.yaml
│   │   └── tests
│   └── values.yaml
├── movie-service
│   ├── Dockerfile
│   ├── app
│   │   ├── api
│   │   │   ├── db.py
│   │   │   ├── db_manager.py
│   │   │   ├── models.py
│   │   │   ├── movies.py
│   │   │   └── service.py
│   │   └── main.py
│   └── requirements.txt
└── nginx_config.conf


Les 4 pipelines s'exécutent correctement depuis le Jenkinsfile

