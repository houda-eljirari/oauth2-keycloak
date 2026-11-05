# Atelier OAuth 2.0 & OpenID Connect (Keycloak + Spring Boot + React)

Ce projet illustre la mise en œuvre d'une architecture de sécurité centralisée basée sur **OAuth 2.0** et **OpenID Connect (OIDC)** avec **Keycloak**, une API backend **Spring Boot** (Resource Server) et un frontend SPA **React** (Client Public avec PKCE).
---

## Architecture du Projet

Le système se compose de trois éléments principaux :

1. **Serveur d'Identité (Keycloak - Port 8080)** : Authentification, émission des jetons JWT, gestion du Realm `enset-realm` et des rôles `USER` / `ADMIN`.
2. **Backend API (Spring Boot - Port 8081)** : Serveur de ressources sécurisé avec validation autonome des signatures JWT et contrôle d'accès basé sur les rôles (RBAC).
3. **Frontend App (React / Vite - Port 5173)** : Single Page Application intégrée avec `keycloak-js`, utilisant le flux *Authorization Code* sécurisé avec **PKCE (S256)**.

---

## Structure du Dépôt

```text
atelier-oauth2/
├── docker-compose.yml          # Orchestration du serveur Keycloak (v26)
├── demandes-api/              # Backend Spring Boot (Resource Server)
│   ├── src/
│   │   └── main/
│   │       ├── java/ma/enset/demandesapi/
│   │       └── resources/application.yml
│   └── pom.xml
└── demandes-front/            # Frontend React + Vite (Client OIDC PKCE)
    ├── src/
    │   ├── api.js             # Client API avec injection automatique du JWT
    │   ├── keycloak.js        # Configuration de l'adaptateur keycloak-js
    │   ├── App.jsx            # Interface utilisateur React et gestion des rôles
    │   └── main.jsx           # Initialisation OIDC avec PKCE (S256)
    ├── package.json
    └── vite.config.js
```

---

## Démarrage Rapide

### 1. Démarrer Keycloak avec Docker
```bash
docker compose up -d
```
* Keycloak sera accessible sur `http://localhost:8080` (Console Admin : `admin` / `admin`).
* Realm configuré : `enset-realm`.
* Utilisateurs de test :
    * `alice` / `alice123` (Rôle: `USER`)
    * `bob` / `bob123` (Rôles: `USER`, `ADMIN`)

### 2. Démarrer l'API Backend Spring Boot
```bash
cd demandes-api
mvn spring-boot:run
```
* L'API démarrera sur `http://localhost:8081`.

### 3. Démarrer le Frontend React
```bash
cd demandes-front
npm install
npm run dev
```
* L'application frontend s'ouvrira sur `http://localhost:5173`.

---

## Fonctionnalités Avancées Implémentées

* **Gestion du statut des demandes** : Consultation, création de demandes par les utilisateurs, et validation / **refus (`PATCH`)** par l'administrateur.
* **Profil utilisateur (`/api/profil`)** : Endpoint dédié dans le backend récupérant dynamiquement l'ID et le nom d'utilisateur directement depuis les claims du jeton JWT.
* **Contrôle d'accès fin (RBAC)** :
    * `/api/public` : Endpoint public.
    * `/api/demandes` : Accessible aux rôles `USER` et `ADMIN`.
    * `/api/admin/**` : Strictement réservé au rôle `ADMIN`.

---

## Tests de Sécurité

Les tests ont été validés avec **Postman** et la console de développement (DevTools) :
* `401 Unauthorized` : Envoi d'une requête sans jeton ou avec un jeton invalide.
* `403 Forbidden` : Tentative d'accès à une route d'administration avec un jeton `USER` (`alice`).
* `200 OK` : Accès autorisé avec un jeton valide et les droits appropriés (`bob`).