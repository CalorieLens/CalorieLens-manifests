# Sealed Secrets

Secrets for CalorieLens are managed with [Bitnami Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets). SealedSecret resources are encrypted with your cluster's public key and are safe to commit to this repository. Only the cluster can decrypt them.

---

## 1. Install the Sealed Secrets Controller

Apply the controller to your cluster (run once):

```bash
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/latest/download/controller.yaml
```

---

## 2. Install the kubeseal CLI

```bash
brew install kubeseal
```

---

## 3. Create and Seal the Secret

The secret named `calorielens-secrets` must contain **all four** of the following keys:

- `DATABASE_URL` — PostgreSQL connection string (asyncpg format), e.g. `postgresql+asyncpg://user:pass@host:5432/calorielens?ssl=disable`
- `OPENAI_API_KEY` — your OpenAI API key (`sk-...`)
- `GOOGLE_CLIENT_ID` — OAuth 2.0 client ID from Google Cloud Console
- `JWT_SECRET_KEY` — random 32+ character secret used to sign JWTs; generate with `openssl rand -hex 32`

Run the following command, substituting the real values:

```bash
kubectl create secret generic calorielens-secrets \
  --from-literal=DATABASE_URL="postgresql+asyncpg://user:pass@host:5432/calorielens?ssl=disable" \
  --from-literal=OPENAI_API_KEY="sk-..." \
  --from-literal=GOOGLE_CLIENT_ID="576451554727-....apps.googleusercontent.com" \
  --from-literal=JWT_SECRET_KEY="$(openssl rand -hex 32)" \
  --namespace calorielens \
  --dry-run=client -o yaml | \
kubeseal --controller-namespace kube-system --format yaml > sealed-secrets/calorielens-secrets.yaml
```

This produces a `SealedSecret` manifest at `sealed-secrets/calorielens-secrets.yaml`.

---

## 4. Commit the Sealed Secret

The output file `calorielens-secrets.yaml` is **safe to commit** — it is encrypted with your cluster's public key and cannot be decrypted outside of your cluster.

```bash
git add sealed-secrets/calorielens-secrets.yaml
git commit -m "chore: add sealed secret for calorielens-secrets"
git push
```

ArgoCD will apply it to the cluster automatically.

---

## 5. Important: Never Commit the Unsealed Secret

Never commit the plain `kubectl create secret` output or any file containing the raw secret values. Only the `SealedSecret` resource (produced by `kubeseal`) is safe to store in git.

Add the following to your `.gitignore` as a safeguard:

```
# Never commit unsealed Kubernetes secrets
*secret*.yaml
!sealed-secrets/calorielens-secrets.yaml
```
