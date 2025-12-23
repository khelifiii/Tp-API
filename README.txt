Étapes pour mettre en place l’API Laravel 12 (JWT, Posts, Rate Limiting)

1) Préparer le projet
   - Cloner ou créer le projet Laravel.
   - Copier .env (ou .env.example) et générer la clé : php artisan key:generate.
   - Choisir la base (ici SQLite par défaut dans .env).

2) Installer JWT
   - composer require php-open-source-saver/jwt-auth:*
   - php artisan vendor:publish --provider="PHPOpenSourceSaver\JWTAuth\Providers\LaravelServiceProvider"
   - php artisan jwt:secret (ajoute JWT_SECRET dans .env)

3) Configurer l’auth JWT
   - Dans config/auth.php : guard par défaut = api, ajouter guard api driver jwt.
   - Modèle User : implémenter JWTSubject et méthodes getJWTIdentifier/getJWTCustomClaims, relation posts().

4) Modèles et migrations
   - Générer Post : php artisan make:model Post -mcr --api.
   - Migration posts : user_id, titre, contenu, statut(brouillon/publie), publie_le, indexes.
   - Modèle Post : fillable, casts, relations (user), scopes publie/brouillon.

5) Contrôleurs & validations
   - AuthController (API) : register, login, me, logout, refresh avec réponses JSON.
   - FormRequests : StorePostRequest, UpdatePostRequest (validation + messages FR).
   - PostController (API) : index (filtres recherche/tri/pagination, visibilité brouillons), store/show/update/destroy protégés par auth JWT.
   - Resource PostResource : formatage JSON cohérent (auteur, dates, statut).

6) Routes API
   - routes/api.php : /auth (register, login) throttlé “auth”; /auth protégé (me, logout, refresh); apiResource posts avec middleware throttle:posts.

7) Rate limiting
   - AppServiceProvider : limiter “api” 60/min, “auth” 5/min (réponses JSON 429), “posts” 20/min authentifié, 10/min invité.

8) Migrations
   - php artisan migrate (création table posts + tables par défaut).

9) Tests rapides
   - Lancer le serveur : php artisan serve.
   - Via Postman/Thunder : register, login, récupérer token, me, refresh, logout.
   - CRUD posts : créer (token), lister (public publié), afficher, mettre à jour, supprimer.
   - Vérifier les réponses 401/403/422 et rate limiting 429.

NOTE ////////////////////////////////////////
Le dossier vendor a été supprimé pour alléger le zip, donc php artisan serve échoue faute de dépendances. Pour relancer :
1) Depuis le dossier du projet :
composer install
2) Si besoin (DB neuve) :
php artisan migrate
3) Démarrer :
php artisan serve
Après composer install, le fichier vendor/autoload.php sera recréé et l’erreur disparaîtra.