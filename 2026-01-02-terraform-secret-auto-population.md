# Terraform Secret Auto-Population Pattern

*Learned: 2026-01-02*

When you have secrets that are derived from other Terraform resources (like Firebase config), you can eliminate manual secret management by having Terraform populate them automatically.

## The Pattern

```hcl
# 1. Create the resource that generates the secret value
resource "google_firebase_web_app" "web" {
  display_name = "My App"
}

# 2. Read the generated config
data "google_firebase_web_app_config" "web" {
  web_app_id = google_firebase_web_app.web.app_id
}

# 3. Auto-populate the secret (use secret_names for full path!)
resource "google_secret_manager_secret_version" "api_key" {
  secret      = module.secret_manager.secret_names["MY_API_KEY"]
  secret_data = data.google_firebase_web_app_config.web.api_key
}
```

## Key Gotcha

`secret_ids` returns just the name (`MY_SECRET`), but `google_secret_manager_secret_version.secret` needs the full resource path (`projects/PROJECT/secrets/MY_SECRET`). Use `secret_names` output instead.

## Benefits

- No manual `gcloud secrets versions add` commands
- Secrets stay in sync when resources change
- Single source of truth in Terraform state

## Sources

- [Terraform google_secret_manager_secret_version](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/secret_manager_secret_version)
