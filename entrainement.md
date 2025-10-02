# Cas de synthèse - Partie 1
Base utilisée " ```tour_Pedia``` ", collection " ```paris``` "

Fichier JSON: tour_Pedia.json

<details open> <summary> Exemples de document : </summary>

```json
{
    "_id" : 83419,
    "contact" : { 
        "GooglePlaces" : null,
        "Foursquare" : "https://fr.foursquare.com/v/pe%C3%B1a-festayre/4adcda06f964a520fd3221e3" 
    }, 
    "name" : "Peña Festayre",
    "location" : {
        "city" : "Paris", 
        "coord" : {
            "coordinates" : [ 2.3860357589657, 48.896621743257 ], 
            "type" : "Point" 
        },
        "address" : "80 Boulevard Macdonald"
    },
    "category" : "restaurant", 
    "description" : "",
    "services" : [ ],
    "reviews" : [ 
        { 
            "wordsCount" : 6, 
            "rating" : 0, 
            "language" : "ca",
            "details" : "http://tour-pedia.org/api/getReviewDetails?id=52a74a85ae9eef5a50671b08",
            "source" : "Foursquare", 
            "text" : "Entree + Plat (grillade a volonte) + dessert 19€",
            "time" : "2011-01-22", "polarity" : 0
        },
        { 
            "wordsCount" : 20, 
            "rating" : 0, 
            "language" : "fr",
            "details" : "http://tour-pedia.org/api/getReviewDetails?id=52a74a85ae9eef5a50671b09",
            "source" : "Foursquare", 
            "text" : "Tous les mercredis jusqu'en juin : Soirées Salsa... concert live puis dj ; 12 femmes, 12 musiciennes, venues des quatre coins du monde",
            "time" : "2012-01-11", 
            "polarity" : 5
        },
        { 
            "wordsCount" : 8, 
            "rating" : 0, 
            "language" : "fr",
            "details" : "http://tour-pedia.org/api/getReviewDetails?id=52a74a85ae9eef5a50671b0a",
            "source" : "Foursquare", 
            "text" : "les soirées salsa sont le jeudi maintenant.",
            "time" : "2012-05-05", 
            "polarity" : 5
        }
    ],
    "likes" : { },
    "nbReviews" : 3
}
```
</details>

Nous appellerons lieux les documents présents dans cette collection. Cette collection contient des lieux de Paris qui ont été agrégés sur le site tourPedia. Vous y trouverez différentes catégories (champ ```category``` et informations pour les lieux :
- des POI (points d'intérêts : ```poi```);
- des restaurants (```restaurant```);
- des logements (```accommodation```), avec les services associés (dans plusieurs langues);
- des attractions (```attraction```);
- à chaque lieu sera associé des commentaires (```reviews```) sur internet (Facebook, Foursquare)

**Remarque : pour connaître le nombre de documents résultat retourné par une requête, faire comme dans l’exemple :** 
```js
var a = db.tour.find({"category":"accommodation"}, {"name":1,"_id":0});
a.itcount();
```
## Requêtes ```find``` ou ```distinct``` - 1 point par requête

### Créer des requêtes simples pour les phrases suivantes :

#### 1. Donner le nom des lieux dont la catégorie est ```accommodation```
Sortie de la requête :
```json
[
    "Forest-Hill Villette",
    "Hôtel Minerve Paris",
    "Hôtel Napoléon",
    "Hôtel Costes",
    "Murano Urban Resort",
    ...
```
**3 318 lieux** avec ```distinct``` ou **3376** sans ```distinct```
<details> <summary>Réponses</summary>

Un seul document avec la liste des noms de lieux (limite : une seule colonne possible) :
```js
db.paris.distinct("name",{"category":{$eq:"accommodation"}})
```
Ou un document par nom de lieu :
```js
db.paris.find({"category":{$eq:"accommodation"}},{"name":1, _id:0})
```
</details>

#### 2. Catégories des lieux ayant au moins une note (```reviews.rating```) de 4 ou plus
Sortie de la requête :
```json
[
    "restaurant",
    "accommodation",
    "attraction",
    "poi"
]
```
<details> <summary>Réponse</summary>

```js
db.paris.distinct("category",{"reviews.rating":{$gte:4}})
```
L'opération ```"reviews.rating" : {"$gte":4 }```, veut dire :
Est-ce que la liste ```reviews``` contient au moins une note supérieure ou égale à 4 ?
</details>

#### 3. Commentaires (```text```) des lieux ayant un commentaire provenant d'une source Facebook
Sortie de la requête :
```json
{
    "name": "Les Princes",
    "reviews": [
        {
            "text" : "Déjeuner au soleil"
        },
        {
            "text" : "Bonne Terrasse"
        },
    ...
```
**353 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {"reviews.rating":{$eq:"Facebook"}}, 
    {"name":1, "reviews.rating":1, "_id":0}
)
```
</details>

#### 4. Donner les noms et notes de lieux ayant au moins une note de 4 ou plus mais aucune note inférieure à 4
Sortie attendue : **5 340 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {"reviews.rating":{$gte:4,$not:{$lt:4}}},
    {"name":1, "reviews.rating":1, _id:0})
```
L'opération ```"reviews.rating" : {"$gte":4,$not:{$lt:4}}```, veut dire :
Est-ce que la liste ```reviews``` contient au moins une note supérieure ou égale à 4 et aucune (```not```) note inférieure à 4 ?
</details>

#### 5. Donner le lieux, la langue et note des commentaires de lieux écrits en français (```fr```) avec une note supérieure à 3. Pour filtrer deux critères sur chaque élément d'une liste : ```$elemMatch```
Sortie attendue : **8 637 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {"reviews":{$elemMatch:{"language":"fr","rating":{$gt:3}}}},
    {"name":1,"reviews.language":1,"reviews.rating":1,"_id":0}
)
```
</details>

#### 6. Nom et services des lieux proposant au moins 1 services
Sortie attendue :
```json
{
    "_id" : NumberInt(83265),
    "name" : "Arès Tour Eiffel",
    "services" : [
        "journaux",
        "centre de remise en forme",
        "petit déjeuner en chambre",
        "réception ouverte 24h 24",
    ...
```
**1 755 lieux**
<details> <summary>Réponses</summary>

```js
db.paris.find(
    {"services.0":{$exists:true,$ne:""}},
    {"name":1,"services":1,"_id":0}
)
```
tableau des services existent ```[]``` et non null et non vide
ou
```js
db.paris.find(
    {"services":{$elemMatch:{$ne:[]}}},
    {"name":1,"services":1,"_id":0}
)
```
</details>

#### 7. Nom et services des lieux proposant au moins 5 services
Sortie attendue : **1 358 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {"services.4":{$exists:true}},
    {"name":1,"services":1}
)
```
on teste s’il existe un indice 4 (5ème élément à partir de l’indice 0) dans le tableau des services
```db.paris.find({"services":{$size :{$gt :4}}}, {name:1,services:1});``` ne fonctionne pas, on ne peut tester que l’égalité entre la taille ```$size``` avec un nombre

</details>

#### 8. Nom et services ayant un service ”```chambres non-fumeurs```” triés par ```_id```
Sortie attendue : **1 313 lieux**
<details> <summary>Réponses</summary>

```js
db.paris.find(
    {"services":"chambres non-fumeurs"},
    {"name":1,"services":1}
).sort({"_id":1})
```
ou
```js
db.paris.createIndex({services:"text"}) // obligatoire sinon $text ne fonctionnne pas
db.paris.find(
    {$text:{$search:"chambres non-fumeurs", $language:"fr",
    $caseSensitive:false, $diacriticSensitive:false}},
    {"name":1,"services":1}
)
```
</details>

#### 9. Nom et services des lieux dont le premier service est ”```chambres non-fumeurs```”
Sortie attendue : **145 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {"services.0":"chambres non-fumeurs"},
    {"name":1,"services":1}
)
```
</details>

#### 10. Nom et services des lieux n'ayant qu'un seul service ”```chambres non-fumeurs```”
Sortie attendue : **15 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {"services":{$size:1,"services.0":"chambres non-fumeurs"}},
    {"name":1,"services":1}
)
```
</details>

#### 11. Donner le nom et numéro de téléphone des lieux ayant un numéro de téléphone renseigné (```$exists```, ```$ne```)
Sortie attendue :
```json
{
    "_id" : NumberInt(83360),
    "contact" : {
        "phone" : "+33 1 43 26 04 16",
    },
    "name" : "Alcar"
}
...
``` 
**50 187 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {"contact.phone":{$exists:true},"contact.phone":{$ne:null}},
    {"name":1,"contact.phone":1}
)
```
</details>

#### 12. Nom et contact Foursquare et website des lieux ayant une URL contact renseignée de type Foursquare et un site web
Sortie attendue : 
```json
{
    "_id" : NmberInt(83622),
    "contact" : {
        "website" : "http://www.zebrasquare.com",
        "GooglePlaces" : null,
        "Foursquare" : "https://fr.foursquare.com/v/zebra-square"
    },
    "name" : "Zebra Square"
}
...
```
**2 070 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {
        "contact.Foursquare":{$exists:true, $ne:null},
        "contact.website":{$exists:true} 
    },
    {name:1,"contact":1}
)
```
</details>

#### 13. Nom des lieux dont le nom contient le mot ”```hotel```” ou ”```Hotel```” ou ”```hôtel```” ou ”```Hôtel```”
Sortie attendue : **1 428 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    { name: { $in: [ /Hotel/, /hotel/ , /hôtel/, /Hôtel/ ] } }, 
    { name:1 }
)
```
</details>

#### 14. Nom et description des lieux ayant une description
Sortie attendue : 
```json 
{
    "_id" : NumberInt(83265),
    "name" : "Arès Tour Eiffel",
    "description" : "L'Hotel Arès Tour Eiffel..."
    ...
```
**1 755 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {description:{$nin : [null, ""] } }, 
    {name:1, description :1}
)
```
</details>

#### 15. Donner les adresses des lieux de catégorie accommodation avec une blanchisserie
Sortie attendue : 
```json 
{
    "_id" : NumberInt(83265),
    "name" : "Arès Tour Eiffel",
    "location" : {
        "address" : "Paris, France, 7 rue...",
    }
}
...
```
**616 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {
        category:"accommodation", 
        services: { $in: ["blanchisserie"] } 
    },
    {name:1,"location.address":1}
)
```
</details>

#### 16. Coordonnées GPS des lieux dont l'adresse contient "```rue de Rome```"
Sortie attendue : 
```json
{
    "_id" : NumberInt(87795),
    "name" : "Villa Eugenie",
    "location" : {
        "coord" : {
            "coordinates" : [ 
                2.3142877221107, 
                48.887155452355 
            ]
        },
        "address" : "Paris, France, 167 rue de Rome, 1..."
    }
}
...
```
**2 lieux**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {"location.address":/rue de Rome/},
    {name:1,"location.address":1,"location.coord.coordinates":1}
)
```
</details>

#### 16 bis. Donner les adresses des lieux de catégorie accommodation avec une blanchisserie et un service de repassage
***(pas dans le sujet)***
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {
        category:"accommodation", 
        services: { $all: ["blanchisserie","service de repassage"] } 
    },
    {name:1,"location.address":1}
)
```
</details>

#### 17. Liste distincte des catégories
Sortie attendue : 
```json
[
    "restaurant",
    "attraction",
    "accommodation",
    "poi"
]
```
<details> <summary>Réponse</summary>

```js
db.paris.distinct("category")
```
</details>

#### 18. Liste distincte des sources pour les reviews
Sortie attendue : 
```json
[
    "Foursquare",
    "GooglePlaces",
    "Facebook",
]
```
<details> <summary>Réponse</summary>

```js
db.paris.distinct("reviews.source")
```
</details>

#### 19. Liste distincte des services (éliminer service nul et vide) 
Sortie attendue : 
```json
[
    "Toutes les parties communes et privées sont non-fumeurs",
    "ascenseur",
    "bagagerie",
    "billeterie",
    "blanchisserie",
    "bureau d'excursions",
    "centre de remise en forme",
    "chambres familiales",
    "chambres non-fumeurs",
    "chauffage",
    "climatisation",
    ...
```
**151 services**
<details> <summary>Réponse</summary>

```js
db.paris.distinct("services",{services:{$ne:null,$gt:""}})
```
</details>

#### 20. Trouver les restaurants à moins de 1 kilomètre du point (2,38 ; 48,89). 
Il faut utiliser les fonctions ```$geoWithin``` et ```$centerSphere```. 

```$centerSphere``` accepte eux paramètres : 
- un tableau de coordonnées ```[ longitude, latitude ]``` 
- une distance en radian. 

Pour convertir une distance en radians, il faut diviser la valeur de distance par lerayon de la sphère (la Terre), où: 3963.2 est le rayon de la Terre en milles. 6378.1 est le rayon de la Terre en km.
Sortie attendue : 
```json
{
    "_id" : NumberInt(83419),
    "name" : "Peña Festayre",
    "location" : {
        "city" : "Paris",
        "address" : "80 Boulevard Macdonald"
    }
}
...
```
**490 restaurants**
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {
        "category":"restaurant", 
        "location.coord.coordinates": { 
            $geoWithin: { 
                $centerSphere: [ 
                    [ 2.38, 48.89 ], 
                    1 / 6378.1 
                ] 
            } 
        }
    }
    ,{name:1,"location.city":1,"location.address":1} )
```
</details>

## Pour les mises à jour suivantes, vérifier le contenu des données avant et après la mise à jour.
#### 1. Mettre à jour tous les lieux dont le nom contient “```montparnasse```” ou “```Montparnasse```” en ajoutant l’attribut ```“area” : “Montparnasse”```
<details> <summary>Réponse</summary>

```js
db.paris.find(
    {$or:[
        {name :/montparnasse/},
        {name:/Montparnasse/}
    ]}
);
db.paris.updateMany(
    {$or:[{name :/montparnasse/},{name:/Montparnasse/}] },
    {$set: {area:"Montparnasse"}}
);
db.paris.find(
    {$or:[
        {name :/montparnasse/},
        {name:/Montparnasse/}
    ]}
);
```
</details>

#### 2. Supprimer tous les lieux n’ayant aucun commentaire
<details> <summary>Réponse</summary>

```js
db.paris.find({reviews:{$size:0}});
db.paris.deleteMany({nbReviews:0})
```
ou
```js
db.paris.find({nbReviews:0});
db.paris.deleteMany({nbReviews:0})
```
</details>

#### 3. Modifier tous les lieux pour ajouter un champ ```wordsCount``` qui correspond à la somme du nombre de mots de tous les commentaires du lieu
<details> <summary>Réponse</summary>

```js
db.paris.aggregate( 
    [
        { $unwind : "$reviews" },
        { $group: { _id : "$_id", nbmots : { $sum : "$reviews.wordsCount" } } },
        { $sort : { _id : 1 } }
    ] 
).forEach( 
    function(x) { 
        db.paris.update( 
            { _id : x._id }, 
            { $set : { wordsCount : x.nbmots} } , 
            { multi : true } 
        ) 
    }
)
```
</details>

## Requêtes avec l’opérateur ```lookup```
Création une collection de données personnalisées **```mycomments```** dans laquelle vous allez y stocker tous vos commentaires sur les destinations que vous avez visité. Pour cela, il faudra y associer l’identifiant du lieu que vous commentez.
<details> <summary> Exemple : </summary>

```js
db.mycomments.save({
    "user":1, 
    "idLocation": 231445, 
    "date":"2017-02-03",
    "comment" : "place magnifique, spacieuse" 
});
```
</details>

#### 1. Créer 5 documents pour la collection ```mycomments``` sur 5 lieux différents (ou non).
<details> <summary>Réponse</summary>

```js
// à compléter
```
</details>

#### 2. Testons la jointure :
```js
db.paris.aggregate([{
    $lookup : {
        "from" : "mycomments",
        "localField":"_id",
        "foreignField" : "idLocation",
        "as" : "mes_infos"
    }
}]);
```
#### 3. Quelle est la taille du résultat produit ?
<details> <summary>Réponse</summary>

```js
db.paris.aggregate( 
    [
        {
            $lookup : { 
                "from" : "mycomments", 
                "localField":"_id", 
                "foreignField" : "idLocation",
                "as" : "mes_infos" 
            }
        },
        { $count:"nombre de documents"} 
    ] 
) // nombre de documents = nombre de documents de la collection « paris »
```
</details>

#### 4. Quelle est la taille du résultat après filtrage sur l’existence de la clé ```mes_infos``` ?
<details> <summary>Réponse</summary>

```js
db.paris.aggregate( 
    [
        {
            $lookup : { 
                "from" : "mycomments", 
                "localField":"_id", 
                "foreignField" : "idLocation",
                "as" : "mes_infos" 
            } 
        },
        { $match: {mes_infos:{$ne:[]}} },
        { $count:"nombre de documents"}
    ] 
) // nombre de documents = nombre de lieux différents de la collection « mycomments »
```
</details>
