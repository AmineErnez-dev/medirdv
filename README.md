# MediRDV : monolithe (React, Express.js, PostgreSQL, Docker)

Application de gestion de rendez-vous médicaux. Deux CRUD : **médecins** et **rendez-vous**.
Structure inspirée de LearnHub, pensée pour être découpée facilement en microservices.

## Lancement

    cp .env.example .env        (déjà fait dans cette archive)
    docker compose up --build

- Application : http://localhost:5173
- API : http://localhost:5000/api/health
- Réinitialiser la base : `docker compose down -v`

## Structure du projet

    medirdv/
    ├── .env.example
    ├── .gitignore
    ├── docker-compose.yml
    ├── backend/
    │   ├── Dockerfile
    │   ├── package.json
    │   └── src/
    │       ├── index.js              # point d'entrée : branche les modules
    │       ├── config.js             # variables d'environnement
    │       ├── events.js             # bus d'événements interne
    │       ├── db/postgres.js        # connexion + création des tables
    │       ├── middleware/
    │       │   └── asyncHandler.js
    │       ├── models/               # requêtes SQL uniquement
    │       │   ├── Doctor.js
    │       │   └── Appointment.js
    │       └── modules/              # routes + logique de chaque CRUD
    │           ├── doctors.js
    │           └── appointments.js
    └── frontend/
        ├── Dockerfile
        ├── package.json
        └── src/
            ├── main.jsx, App.jsx
            ├── pages/        # Home, Doctors, Appointments
            ├── components/   # Layout, Navbar, ConfirmDialog, EmptyState
            ├── context/      # ToastContext
            ├── hooks/        # useFetch
            ├── services/     # api.js
            ├── styles/       # theme.css, app.css
            └── utils/        # format.js

## Règles pour garder les modules indépendants

1. `modules/appointments.js` ne lit jamais la table `doctors` : pas de JOIN, pas de clé étrangère.
2. Le seul point de contact avec les médecins est le bloc `findDoctor / findDoctors` en haut du fichier.
3. Les réactions entre modules passent par `events` (`doctor.deleted` annule les rendez-vous du médecin).

## Référence de l'API

| Méthode | Route | Rôle |
|---|---|---|
| GET | /api/health | santé de l'API |
| GET / POST | /api/doctors | lister / ajouter un médecin |
| GET / PUT / DELETE | /api/doctors/:id | détail / modifier / supprimer |
| GET / POST | /api/appointments | lister / réserver |
| GET / PUT / DELETE | /api/appointments/:id | détail / modifier / supprimer |

## Passage en microservices

| Monolithe | Microservice |
|---|---|
| `modules/doctors.js` + `models/Doctor.js` | service Médecins |
| `modules/appointments.js` + `models/Appointment.js` | service Rendez-vous |
| table `doctors` / table `appointments` | `doctors_db` / `appointments_db` |
| `events.js` | RabbitMQ (`emit` devient publish, `on` devient consume) |
| `findDoctor / findDoctors` | appel HTTP ou cache local alimenté par événements |
| routes `/api/...` de `index.js` | API Gateway |
