# Cloud Architecture Blog (Hugo + GitHub Pages)

Blog bilingüe ES/EN sobre arquitectura en AWS, Azure y Google Cloud. Tema propio, sin dependencias externas ni submódulos.
Bilingual ES/EN blog on AWS, Azure and Google Cloud architecture. Custom theme, no external themes or submodules.

---

## ES — Puesta en marcha

### 1. Requisitos
- Hugo **extended** ≥ 0.158 (probado con 0.167.0): `winget install Hugo.Hugo.Extended`
- Git y GitHub CLI: `winget install GitHub.cli`

### 2. Probar en local
```bash
hugo server -D
# http://localhost:1313  (ES)  y  http://localhost:1313/en/  (EN)
```

### 3. Crear el repo y activar Pages con GitHub Actions
```bash
gh auth login
gh repo create cloud-blog --public --source=. --push
gh api -X POST repos/{owner}/cloud-blog/pages -f build_type=workflow
```
Cada push a `main` compila y publica mediante `.github/workflows/hugo.yml`.

### 4. Dominio propio
1. `baseURL` en `hugo.toml` ya apunta a `https://www.dggcloud.com/`.
2. Crea los registros DNS de `dggcloud.com` (subdominio `www` + dominio raíz, para que GitHub redirija `dggcloud.com` → `www.dggcloud.com`):

   | Nombre | Tipo | Valor |
   |---|---|---|
   | `www.dggcloud.com` | CNAME | `TU_USUARIO.github.io` |
   | `dggcloud.com` | A | `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153` |
   | `dggcloud.com` | AAAA | `2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153`, `2606:50c0:8003::153` |

3. Configura el dominio en el repo:
   ```bash
   gh api -X PUT repos/{owner}/cloud-blog/pages -f cname=www.dggcloud.com
   ```
4. Cuando GitHub emita el certificado (puede tardar hasta 24 h), fuerza HTTPS:
   ```bash
   gh api -X PUT repos/{owner}/cloud-blog/pages -F https_enforced=true
   ```
5. **Verifica el dominio** en *Settings → Pages* de tu cuenta (registro TXT) para evitar domain takeover.

> Con despliegue por GitHub Actions el archivo `CNAME` se ignora: el dominio se configura en los ajustes de Pages (paso 3).

Ejemplo si el DNS de `dggcloud.com` está en Route 53:
```bash
aws route53 change-resource-record-sets --hosted-zone-id ZONE_ID --change-batch '{
  "Changes": [
    {"Action": "UPSERT", "ResourceRecordSet": {"Name": "www.dggcloud.com", "Type": "CNAME", "TTL": 300,
      "ResourceRecords": [{"Value": "TU_USUARIO.github.io"}]}},
    {"Action": "UPSERT", "ResourceRecordSet": {"Name": "dggcloud.com", "Type": "A", "TTL": 300,
      "ResourceRecords": [{"Value": "185.199.108.153"}, {"Value": "185.199.109.153"}, {"Value": "185.199.110.153"}, {"Value": "185.199.111.153"}]}},
    {"Action": "UPSERT", "ResourceRecordSet": {"Name": "dggcloud.com", "Type": "AAAA", "TTL": 300,
      "ResourceRecords": [{"Value": "2606:50c0:8000::153"}, {"Value": "2606:50c0:8001::153"}, {"Value": "2606:50c0:8002::153"}, {"Value": "2606:50c0:8003::153"}]}}
  ]}'
```

Comprobación / Check:
```bash
nslookup www.dggcloud.com
nslookup dggcloud.com
```

### 5. Escribir un artículo
```bash
hugo new content posts/mi-articulo.es.md
hugo new content posts/mi-articulo.en.md
```
- Usa el mismo nombre de archivo con sufijo `.es.md` / `.en.md` para enlazar las traducciones.
- `clouds: [aws, azure, gcp]` controla la franja de colores y las páginas por proveedor.
- Quita `draft: true` para publicar.
- Diagramas: bloques de código ```` ```mermaid ````. Tabla de contenidos automática en artículos de más de 600 palabras (`toc: false` para desactivarla).

---

## EN — Getting started

### 1. Requirements
- Hugo **extended** ≥ 0.158 (tested with 0.167.0), Git, GitHub CLI.

### 2. Run locally
```bash
hugo server -D
```

### 3. Create the repo and enable Pages via GitHub Actions
```bash
gh repo create cloud-blog --public --source=. --push
gh api -X POST repos/{owner}/cloud-blog/pages -f build_type=workflow
```

### 4. Custom domain
1. `baseURL` in `hugo.toml` is already `https://www.dggcloud.com/`.
2. DNS: `www.dggcloud.com` `CNAME` → `YOUR_USER.github.io`, plus apex `dggcloud.com` `A` records `185.199.108.153`–`185.199.111.153` and `AAAA` `2606:50c0:8000::153`–`8003::153`, so GitHub redirects the apex to `www`.
3. `gh api -X PUT repos/{owner}/cloud-blog/pages -f cname=www.dggcloud.com`
4. Once the certificate is issued: `gh api -X PUT repos/{owner}/cloud-blog/pages -F https_enforced=true`
5. Verify the domain in your account's Pages settings (TXT record) to prevent takeovers.

> When deploying with GitHub Actions, a `CNAME` file is ignored; the domain lives in the Pages settings.

### 5. Writing posts
Same filename with `.es.md` / `.en.md` suffix links translations. `clouds:` drives the provider strip and per-cloud pages. Use ```` ```mermaid ```` blocks for diagrams.

---

## Estructura / Structure
```
hugo.toml                 Config: idiomas, menús, taxonomías / languages, menus, taxonomies
content/posts/            Artículos (*.es.md / *.en.md)
content/clouds/           Páginas de AWS, Azure, GCP
layouts/                  Plantillas (sistema de plantillas de Hugo ≥ 0.146)
assets/css/               main.css + syntax.css (claro/oscuro)
i18n/                     Textos de interfaz ES/EN
.github/workflows/        Despliegue a GitHub Pages
```

## Referencias / References
- GitHub Pages custom domains: https://docs.github.com/pages/configuring-a-custom-domain-for-your-github-pages-site
- Pages publishing with Actions: https://docs.github.com/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
- Hugo on GitHub Pages: https://gohugo.io/host-and-deploy/host-on-github-pages/
- Hugo multilingual: https://gohugo.io/content-management/multilingual/
