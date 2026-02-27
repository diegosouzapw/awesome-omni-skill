---
name: r2-transfer-service-playbook
description: Manage changes to the R2 transfer pipeline (Python service, Cloudflare Workers, PHP logger) with mandatory validations, allowlists, and regression checks.
---

# R2 Transfer Service Playbook

## Portée
Utilise ce skill lorsqu'une tâche touche :
- `services/r2_transfer_service.py`
- `email_processing/orchestrator.py` (section delivery_links / r2_url)
- `deployment/cloudflare-worker/*`
- `deployment/public_html/config_api.php`, `deployment/public_html/test-direct.php`, pages de test R2

## Pré-requis
- ENV obligatoires : `R2_BASE_URL`, `R2_FETCH_TOKEN`, `ALLOWED_R2_DOMAINS`.
- Virtualenv `/mnt/venv_ext4/venv_render_signal_server` pour les scripts/tests.
- Accès au déploiement Cloudflare Workers (wrangler).

## Workflow
1. **Analyse des dépendances**
   - Confirmer la présence des ENV ci-dessus.
   - Vérifier les allowlists (Dropbox/FromSmash/SwissTransfer) avant toute nouvelle source.
2. **Mises à jour Python**
   - Toujours valider les domaines via `is_allowed_domain`.
   - Injecter le header `X-R2-FETCH-TOKEN` pour chaque requête Worker.
   - Conserver les logs sans PII et retour fallback `raw_url` en cas d'échec.
3. **Workers Cloudflare**
   - Garder le mode fetch avec timeout 120s pour Dropbox `/scl/fo/`.
   - Mettre à jour `httpMetadata.contentDisposition` pour préserver le nom de fichier.
   - Exécuter `wrangler deploy --dry-run` (documenter la sortie).
4. **PHP Logger / Diagnostics**
   - Assurer que `config_api.php` continue d'écrire les paires `source_url`/`r2_url`.
   - Tester `deployment/public_html/test-direct.php` pour valider le flux complet.
5. **Tests & validation**
   - Lancer le helper `./.cline/skills/r2-transfer-service-playbook/test_r2_worker.sh`.
   - Compléter si besoin avec des tests ciblés sur les nouvelles sources.
6. **Documentation & Memory Bank**
   - Mettre à jour `docs/processing/file-offload.md` ou section dédiée R2.
   - Ajouter une entrée dans la Memory Bank si de nouvelles sources sont supportées.

## Ressources
- `test_r2_worker.sh` : active le venv, exécute les tests R2 et vérifie la page de test PHP.

## Conseils
- En cas de nouvelle plateforme de fichiers, créer un helper dédié (normalisation URL, validation).
- Limiter les retries côté Worker, gérer les erreurs HTTP explicitement.
- Garder les scripts de cleanup alignés (expiration 24h par défaut).
