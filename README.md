Electronic -shop 
web application for buying electronic product 
## Architecture

- Frontend: Next.js + TypeScript
- Backend: Spring Boot + Java
- Database: PostgreSQL

## Project Structure

- `frontend/` → Next.js application
- `backend/` → Spring Boot REST API


## 📡 Endpoints principaux

### Authentification — `/api/auth`
| Méthode | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/api/auth/register` | Inscription client/vendeur | Public |
| POST | `/api/auth/login` | Connexion → access + refresh token | Public |
| POST | `/api/auth/refresh` | Renouveler l'access token | Refresh token |
| POST | `/api/auth/logout` | Invalidation du refresh token | Bearer |

### Produits — `/api/products`
| Méthode | Endpoint | Description | Auth |
|---|---|---|---|
| GET | `/api/products` | Liste paginée + filtres | Public |
| GET | `/api/products/{id}` | Détail + variantes + avis | Public |
| POST | `/api/products` | Créer un produit | SELLER/ADMIN |
| PUT | `/api/products/{id}` | Modifier un produit | SELLER/ADMIN |
| DELETE | `/api/products/{id}` | Désactiver (soft delete) | SELLER/ADMIN |
| GET | `/api/products/search?q=` | Recherche full-text | Public |
| GET | `/api/products/top-selling` | Top 10 meilleures ventes | Public |

### Panier — `/api/cart`
| Méthode | Endpoint | Description | Auth |
|---|---|---|---|
| GET | `/api/cart` | Panier du client connecté | CUSTOMER |
| POST | `/api/cart/items` | Ajouter un article | CUSTOMER |
| PUT | `/api/cart/items/{id}` | Modifier quantité | CUSTOMER |
| DELETE | `/api/cart/items/{id}` | Retirer un article | CUSTOMER |
| POST | `/api/cart/coupon` | Appliquer un coupon | CUSTOMER |
| DELETE | `/api/cart/coupon` | Retirer le coupon | CUSTOMER |

### Commandes — `/api/orders`
| Méthode | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/api/orders` | Passer une commande depuis le panier | CUSTOMER |
| GET | `/api/orders/my` | Mes commandes | CUSTOMER |
| GET | `/api/orders/{id}` | Détail d'une commande | CUSTOMER/SELLER/ADMIN |
| PUT | `/api/orders/{id}/status` | Mettre à jour le statut | SELLER/ADMIN |
| PUT | `/api/orders/{id}/cancel` | Annuler une commande | CUSTOMER/ADMIN |
| GET | `/api/orders` | Toutes les commandes | ADMIN |

### Avis — `/api/reviews`
| Méthode | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/api/reviews` | Poster un avis (achat vérifié) | CUSTOMER |
| GET | `/api/reviews/product/{id}` | Avis d'un produit | Public |
| PUT | `/api/reviews/{id}/approve` | Approuver un avis | ADMIN |

### Coupons — `/api/coupons`
| Méthode | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/api/coupons` | Créer un coupon | ADMIN |
| PUT | `/api/coupons/{id}` | Modifier un coupon | ADMIN |
| DELETE | `/api/coupons/{id}` | Supprimer un coupon | ADMIN |
| GET | `/api/coupons/validate/{code}` | Vérifier la validité | CUSTOMER |

### Dashboard — `/api/dashboard`
| Méthode | Endpoint | Description | Auth |
|---|---|---|---|
| GET | `/api/dashboard/admin` | Stats globales (CA, top produits...) | ADMIN |
| GET | `/api/dashboard/seller` | Stats vendeur connecté | SELLER |