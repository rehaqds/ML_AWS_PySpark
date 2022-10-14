# Déploiement d'un modèle dans le cloud


## 1. Le contexte

Le but de ce projet est de créer un classifieur d'images de fruits et légumes qui sera utilisé par des robots cueilleurs. L'outil devra travailler dans un environnement Big Data sur AWS et avec PySpark pour distribuer les calculs. Afin de limiter les frais, seuls 2 serveurs EC2 sont utilisés et le nombre d'images est limité.

L'étude est effectuée sur AWS EMR. Les données sont divisées en paquets et envoyées à plusieurs ordinateurs qui effectuent en parallèle les différentes transformations/actions en gardant les données dans leur RAM.


## 2. Les données

Le jeu de données d’images de fruits et légumes est "Fruits 360" (Muresan & Oltean, 2018).

131 variétés. Photos 360° triaxiales post-traitées (100x100 pixels, fond blanc).

![image](https://user-images.githubusercontent.com/108366684/195697402-acb41ddd-9425-4378-a1fe-9550d4481dde.png)

68k fichiers train + 22k test → 5x10 = 50 images seulement sont utilisées afin de limiter le coût d'AWS.


## 3. Architecture Big Data mise en place

![image](https://user-images.githubusercontent.com/108366684/195698097-e7e7b15b-896c-4e6d-80da-a12785040081.png)


## 4. Modélisation

Les calculs sont distribués sur 4 partitions : [(0, 13), (1, 13), (2, 13), (3, 11)]

On utilise le principe de transfer learning en retirant la dernière couche d’un CNN MobileNet (entraîné sur ImageNet) pour obtenir 1280 features par image → PandasUDF (Arrow) pour pouvoir utiliser directement un modèle CNN de TensorFlow avec calculs distribués.
Une architecture MobileNet est utilisée ici en raison du "faible" nombre de paramètres dans le but de limiter le coût des calculs sur AWS.

Ensuite on effectue une réduction de dimension : ACP (18 composantes = 90% variance expliquée) après un standard scaling → pyspark.ml

![image](https://user-images.githubusercontent.com/108366684/195698981-176b1493-eb95-4553-a3ea-f1b40a7b881c.png)


## 5. Conclusion

Cette première étape a donc permis de mettre en place l’infrastructure Big Data nécessaire au futur développement du projet de robot cueilleur.

Travail complémentaire à effectuer par la suite (si financement disponible pour AWS):
- Tester différents modèles de classification en appliquant un fine-tuning sur plusieurs algorithmes CNN pré-entraînés (EfficientNet, Xception, ...).
- Augmenter la taille/puissance du cluster utilisé sur le Cloud afin de pouvoir travailler avec l’intégralité du jeu de données (90k) puis des datasets plus volumineux et plus variés.

