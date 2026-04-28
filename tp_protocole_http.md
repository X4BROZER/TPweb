# Chapitre 6 : Le Protocole HTTP — Travaux Pratiques


---

## TP 1 : Exploration avec les DevTools

### Objectifs

- Analyser les requêtes/réponses HTTP
- Comprendre les headers
- Observer les codes de statut

### 1.1 Ouvrir les DevTools

1. Ouvrez Chrome ou Firefox
2. Appuyez sur **F12** ou **Ctrl+Shift+I**
3. Allez dans l'onglet **Network** (Réseau)

### 1.2 Observer une requête simple

1. Cochez "Preserve log" pour garder l'historique
2. Naviguez vers `https://github.com`
3. Cliquez sur la requête dans la liste

**Réponse observée sur `https://github.com` :**

```
HTTP/2 200
date: Tue, 28 Apr 2026 21:39:04 GMT
content-type: text/html; charset=utf-8
etag: W/"79296996b057282f2a2ffb7e6d3207aa"
cache-control: max-age=0, private, must-revalidate
server: github.com
```

**Réponses aux questions :**

- **Code de statut :** `200 OK`
- **Headers de requête envoyés :** `Host`, `User-Agent`, `Accept`, `Accept-Encoding`, `Connection`
- **Content-Type de la réponse :** `text/html; charset=utf-8`

### 1.3 Tester différentes méthodes

```javascript
// GET
fetch('https://httpbin.org/get')
  .then(r => r.json())
  .then(console.log);

// POST
fetch('https://httpbin.org/post', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'John', age: 30 })
})
  .then(r => r.json())
  .then(console.log);
```

### 1.4 Observer les codes de statut

**Tableau des résultats :**

| URL                          | Méthode | Code attendu | Signification               |
|------------------------------|---------|--------------|------------------------------|
| `httpbin.org/status/200`     | GET     | `200`        | OK                           |
| `httpbin.org/status/404`     | GET     | `404`        | Not Found                    |
| `httpbin.org/status/500`     | GET     | `500`        | Internal Server Error        |
| `httpbin.org/redirect/3`     | GET     | `302 → 200`  | 3 redirections puis OK       |

### Exercice — Tableau complété

| URL                      | Méthode | Code | Content-Type                    |
|--------------------------|---------|------|---------------------------------|
| `httpbin.org/get`        | GET     | 200  | `application/json`              |
| `httpbin.org/post`       | POST    | 200  | `application/json`              |
| `httpbin.org/status/201` | GET     | 201  | `text/plain; charset=utf-8`     |

---

## TP 2 : Maîtrise de cURL

### Objectifs

- Utiliser cURL en ligne de commande
- Comprendre les options principales
- Envoyer différents types de requêtes

### 2.1 Requête GET simple

```bash
curl https://github.com           # Corps HTML uniquement
curl -i https://github.com        # Headers + Corps
curl -v https://github.com        # Mode debug complet
```

**Réponse à la question — Différence entre `-i` et `-v` :**

| Option | Comportement |
|--------|-------------|
| `-i`   | Affiche les **headers de réponse** puis le corps. Usage courant pour voir le statut et les headers. |
| `-v`   | Mode **verbose/debug** : affiche la négociation TLS, les headers de *requête ET réponse*, très verbeux. |

**Résultat réel `curl -i https://github.com` (extrait) :**

```
HTTP/2 200
date: Tue, 28 Apr 2026 21:38:52 GMT
content-type: text/html; charset=utf-8
cache-control: max-age=0, private, must-revalidate
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
referrer-policy: origin-when-cross-origin, strict-origin-when-cross-origin
```

### 2.2 Requête POST avec données

```bash
# Form data
curl -X POST -d "name=John&email=john@example.com" \
  https://httpbin.org/post

# JSON
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "email": "john@example.com"}' \
  https://httpbin.org/post
```

### 2.3 Headers personnalisés

```bash
curl -H "Authorization: Bearer mon-token-secret" \
     -H "Accept: application/json" \
     https://httpbin.org/headers
```

### 2.4 Suivre les redirections

**Résultats réels sur `https://github.com` :**

```
Sans -L : Code=200
Avec -L  : Code=200, URL finale=https://github.com/
```

```bash
curl https://httpbin.org/redirect/3        # S'arrête à la 1re redirection (302)
curl -L https://httpbin.org/redirect/3     # Suit toutes les redirections → 200 final
```

### 2.5 Télécharger un fichier

```bash
curl -o image.png https://httpbin.org/image/png
curl -O https://example.com/fichier.pdf
```

### Exercice avancé — Solution

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-Custom-Header: MonHeader" \
  -d '{"action": "test", "value": 42}' \
  -i \
  https://httpbin.org/post
```

---

## TP 3 : API REST avec JavaScript

### Objectifs

- Utiliser l'API Fetch
- Gérer les promesses et async/await
- Traiter les erreurs HTTP

### 3.1 GET basique

```javascript
async function getUsers() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/users');
    const users = await response.json();
    console.log(users);
  } catch (error) {
    console.error('Erreur:', error);
  }
}
```

**Résultat attendu (extrait du 1er utilisateur) :**

```json
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
  "phone": "1-770-736-0986 x56442",
  "website": "hildegard.org"
}
```

### 3.2 POST — Créer une ressource

```javascript
async function createPost(data) {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

createPost({ title: 'Mon article', body: "Contenu de l'article", userId: 1 })
  .then(console.log);
```

**Résultat obtenu :**

```json
{
  "title": "Mon article",
  "body": "Contenu de l'article",
  "userId": 1,
  "id": 101
}
```

> Code de statut retourné : **`201 Created`**

### 3.3 PUT — Modifier une ressource

```javascript
async function updatePost(id, data) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  return response.json();
}
```

**Résultat obtenu pour `updatePost(1, { title: "Titre modifié", ... })` :**

```json
{
  "id": 1,
  "title": "Titre modifié",
  "body": "Nouveau contenu",
  "userId": 1
}
```

> Code de statut : **`200 OK`**

### 3.4 DELETE — Supprimer une ressource

```javascript
async function deletePost(id) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
    method: 'DELETE'
  });
  return response.ok;
}
```

**Résultat obtenu pour `deletePost(1)` :**

```
Code de statut : 200 OK
Corps de réponse : {}
Retour de la fonction : true
```

### Exercice pratique — Solution `fetchWithRetry`

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      if (response.status >= 500 && attempt < maxRetries) {
        console.warn(`Tentative ${attempt} échouée (${response.status}), retry...`);
        await new Promise(resolve => setTimeout(resolve, 1000));
        continue;
      }
      return response;
    } catch (err) {
      if (attempt === maxRetries) throw err;
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }
}
```

---

## TP 4 : Analyse des Headers de Sécurité

### Objectifs

- Comprendre les headers de sécurité
- Analyser la configuration d'un site
- Identifier les améliorations possibles

### 4.1 Commandes utilisées

```bash
curl -I https://github.com | grep -i "strict\|x-frame\|x-content\|content-security\|referrer"
```

### Headers à vérifier

| Header                      | But                   | Valeur recommandée                         |
|-----------------------------|-----------------------|--------------------------------------------|
| `Strict-Transport-Security` | Forcer HTTPS          | `max-age=31536000; includeSubDomains`      |
| `X-Frame-Options`           | Anti-clickjacking     | `DENY` ou `SAMEORIGIN`                     |
| `X-Content-Type-Options`    | Anti-MIME sniffing    | `nosniff`                                  |
| `Content-Security-Policy`   | Sources autorisées    | `default-src 'self'`                       |
| `Referrer-Policy`           | Contrôle du referrer  | `strict-origin-when-cross-origin`          |

### Exercice — Résultats réels (28 avril 2026)

| Site         | HSTS                                          | X-Frame-Options | X-Content-Type | Referrer-Policy                                               | Note              |
|--------------|-----------------------------------------------|-----------------|----------------|---------------------------------------------------------------|-------------------|
| `github.com` | `max-age=31536000; includeSubdomains; preload` | `deny`          | `nosniff`      | `origin-when-cross-origin, strict-origin-when-cross-origin`   | ✅ A+ (excellent) |
| `google.com` | présent (standard)                            | `SAMEORIGIN`    | `nosniff`      | `no-referrer`                                                 | ✅ Bon            |
| `daaif.net`  | absent                                        | absent          | absent         | absent                                                        | ⚠️ À améliorer   |

**Détail complet des headers de sécurité de `github.com` :**

```
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
x-xss-protection: 0
referrer-policy: origin-when-cross-origin, strict-origin-when-cross-origin
content-security-policy:
  default-src 'none';
  base-uri 'self';
  script-src github.githubassets.com;
  style-src 'unsafe-inline' github.githubassets.com;
  img-src 'self' data: blob: github.githubassets.com ...;
  connect-src 'self' uploads.github.com api.github.com ...;
  frame-ancestors 'none';
  upgrade-insecure-requests;
```

> **Analyse :** GitHub obtient un score **A+**. Tous les headers critiques sont présents. La CSP est stricte avec `default-src 'none'` et `frame-ancestors 'none'`. Le header `x-xss-protection: 0` est désactivé volontairement car les navigateurs modernes utilisent la CSP à la place.

---

## TP 5 : Cache HTTP

### Objectifs

- Comprendre le fonctionnement du cache
- Observer les headers de cache
- Tester la validation avec ETag

### 5.1 Observer le cache

```bash
curl -i https://httpbin.org/cache/60
# Vérifie : Cache-Control, ETag, Expires
```

**Résultats réels des headers de cache sur `github.com` :**

```
etag: W/"968f544a42a3928aadf0421808e644a2"
cache-control: max-age=0, private, must-revalidate
set-cookie: logged_in=no; expires=Wed, 28 Apr 2027 21:39:00 GMT; HttpOnly; Secure; SameSite=Lax
```

**Interprétation :**

| Header          | Valeur                              | Signification                                                        |
|-----------------|-------------------------------------|----------------------------------------------------------------------|
| `etag`          | `W/"968f544a..."` (ETag faible)     | Identifiant de version, utilisé pour valider le cache                |
| `cache-control` | `max-age=0, private, must-revalidate` | Pas de cache partagé, revalider à chaque requête                   |
| `set-cookie`    | `HttpOnly; Secure; SameSite=Lax`   | Cookie de session sécurisé, inaccessible via JavaScript              |

### 5.2 Requête conditionnelle avec ETag

```bash
# Étape 1 : obtenir l'ETag
curl -i https://httpbin.org/etag/test123
# → ETag: "test123", code 200

# Étape 2 : requête conditionnelle
curl -i -H "If-None-Match: test123" https://httpbin.org/etag/test123
# → 304 Not Modified (corps vide → économie de bande passante)
```

**Schéma du fonctionnement :**

```
1re requête :
  Client → Serveur : GET /resource
  Serveur → Client : 200 OK + ETag: "abc123" + corps complet

2e requête (cache conditionnel) :
  Client → Serveur : GET /resource + If-None-Match: "abc123"
  Serveur → Client : 304 Not Modified  (pas de corps)
```

### 5.3 Simulation de cache dans le navigateur

| Action              | Comportement observé                                          |
|---------------------|---------------------------------------------------------------|
| Premier chargement  | Toutes les ressources chargées depuis le réseau               |
| **F5**              | Certaines ressources : `(from cache)` ou `304 Not Modified`   |
| **Ctrl+Shift+R**    | Rechargement forcé — toutes les ressources : `200 OK`         |

### Exercice — Configuration des headers de cache par type de fichier

| Type de fichier | Header recommandé                                        | Raison                                        |
|-----------------|----------------------------------------------------------|-----------------------------------------------|
| HTML            | `Cache-Control: no-cache, must-revalidate`               | Toujours revalider pour avoir le contenu frais |
| CSS / JS        | `Cache-Control: public, max-age=31536000, immutable`     | Fichiers versionnés, cachables 1 an           |
| Images          | `Cache-Control: public, max-age=86400`                   | Cachables 24h                                 |

---

## Exercices Récapitulatifs

### Exercice 1 : Client HTTP minimaliste — Solution

```javascript
async function sendRequest(url, method, body) {
  const options = { method, headers: { 'Content-Type': 'application/json' } };
  if (body && method !== 'GET') options.body = JSON.stringify(JSON.parse(body));

  const response = await fetch(url, options);
  const headers = [...response.headers.entries()];
  const text = await response.text();

  return {
    status: `${response.status} ${response.statusText}`,
    headers,
    body: text
  };
}
```

### Exercice 2 : Réponses aux questions théoriques

**1. Quelle est la différence entre `no-cache` et `no-store` ?**

| Directive  | Comportement                                                                                    |
|------------|--------------------------------------------------------------------------------------------------|
| `no-cache` | La ressource **peut** être mise en cache, mais doit être **revalidée** avant chaque utilisation (via ETag/Last-Modified). |
| `no-store` | La ressource **ne doit jamais** être stockée, ni en cache navigateur, ni en cache intermédiaire. Utilisé pour les données sensibles (ex : données bancaires). |

**2. Pourquoi POST n'est-il pas idempotent ?**

Envoyer plusieurs fois la même requête POST crée plusieurs ressources distinctes (ex : plusieurs commandes). À l'inverse, `PUT` et `DELETE` sont idempotents : répéter l'opération produit le même résultat.

**3. Que se passe-t-il si le serveur renvoie un code 301 ?**

Le navigateur reçoit une redirection **permanente** vers une nouvelle URL (header `Location`). Il mémorise cette redirection en cache et redirige automatiquement les futures requêtes vers la nouvelle URL sans reconsulter l'ancienne.

**4. À quoi sert le header `Origin` ?**

Il indique l'origine (schéma + domaine + port) de la requête. Envoyé automatiquement pour les requêtes cross-origin, il sert au mécanisme **CORS** : le serveur autorise ou refuse selon l'origine déclarée.

**5. Pourquoi utiliser `HttpOnly` sur les cookies de session ?**

Le flag `HttpOnly` empêche JavaScript d'accéder au cookie via `document.cookie`. Cela protège contre les attaques **XSS** : même si un attaquant injecte du code malveillant, il ne peut pas voler le cookie de session.

---

