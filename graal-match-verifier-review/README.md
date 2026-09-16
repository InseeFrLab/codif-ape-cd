# graal-match-verifier-review

App Flask de revue humaine des verdicts du *MatchVerifier*, utilisée par les
annotateurs du projet GRAAL. Source, image et manifests d'origine :
[InseeFrLab/GRAAL](https://github.com/InseeFrLab/GRAAL) —
[`src/evaluation/apps/match_verifier_eval_app.py`](https://github.com/InseeFrLab/GRAAL/blob/main/src/evaluation/apps/match_verifier_eval_app.py).

- Image : `meilametayebjee/graal-match-verifier-review`, poussée par la CI de GRAAL à
  chaque push sur `main`. Épingler un tag `sha-xxxxxxx` plutôt que `latest` pour
  figer la version pendant une campagne de revue.
- URL : <https://graal-match-verifier-review.lab.sspcloud.fr>
- Secrets attendus dans le namespace : `my-s3-creds` (déjà présent) et
  `graal-match-verifier-review`, à créer une fois :

  ```bash
  kubectl create secret generic graal-match-verifier-review \
    --from-literal=NEO4J_URL=... \
    --from-literal=NEO4J_USERNAME=... \
    --from-literal=NEO4J_PWD=... \
    --from-literal=EMBEDDING_MODEL=... \
    --from-literal=URL_EMBEDDING_API=... \
    --from-literal=OPENAI_API_KEY=...
  ```

  Il est `optional: true` : sans lui l'app démarre quand même, la connexion Neo4j
  échoue proprement et la revue se fait sans les notices de code.

Deux invariants à ne pas casser :

- `replicas: 1` et `strategy: Recreate`. Les jugements sont ajoutés à un JSONL
  unique sur S3, et un append S3 réécrit l'objet entier : deux pods concurrents
  perdraient des jugements.
- Les arguments `--input`, `--reviewers`, `--shared-n`, `--unique-n` et `--seed`
  déterminent quelles lignes vont à quel annotateur. Les changer en cours de
  campagne redistribue les paquets sous les pieds des annotateurs. `--commit` et
  `--model`, eux, désignent le run revu et sont faits pour être changés d'une
  campagne à l'autre.
