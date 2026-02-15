# allio-updates

Repositorio central de actualizaciones para **Allio Client**.

Contiene los manifiestos de actualización del launcher y los archivos de cada instancia
(mods, configs, resourcepacks) servidos vía `raw.githubusercontent.com`.

## Estructura

```
allio-updates/
├── launcher/
│   └── betatest/
│       └── update_manifest.json      ← manifest del launcher (versión, url ZIP, sha256)
├── instances/
│   ├── betatest/
│   │   ├── manifest.json             ← manifest de instancia (lista de archivos + hashes)
│   │   ├── mods/                     ← JARs de mods
│   │   ├── config/                   ← archivos de configuración
│   │   └── resourcepacks/            ← resource packs
│   ├── permadeath/
│   │   ├── manifest.json
│   │   ├── mods/
│   │   ├── config/
│   │   └── resourcepacks/
│   └── squidgame/
│       ├── manifest.json
│       ├── mods/
│       ├── config/
│       └── resourcepacks/
├── .gitattributes
└── README.md
```

## Cómo funciona

1. El **launcher** (mainbetatest.py) al iniciar consulta:
   ```
   https://raw.githubusercontent.com/<owner>/allio-updates/main/launcher/betatest/update_manifest.json
   ```
   Si hay versión nueva → descarga ZIP → se auto-actualiza.

2. Al entrar a una instancia, consulta:
   ```
   https://raw.githubusercontent.com/<owner>/allio-updates/main/instances/betatest/manifest.json
   ```
   Compara SHA256 de cada archivo local vs remoto → descarga solo los que cambiaron.

## Workflow de publicación

### Actualizar mods/config de una instancia

```powershell
# Desde la raíz del proyecto Allio-Client
python sync_to_updates_repo.py --instance betatest
# Esto copia mods/config/resourcepacks al repo local y regenera manifest.json

cd allio-updates
git add -A
git commit -m "chore: update betatest mods v1.0.15"
git push origin main
```

### Publicar nueva versión del launcher

1. Compilar con PyInstaller → ZIP del launcher
2. Subir ZIP como asset de un GitHub Release (repo `allio-instances`)
3. Actualizar `launcher/betatest/update_manifest.json` con nueva versión + URL + SHA256
4. Push a `allio-updates`

## Configuración inicial

```bash
# Crear el repo en GitHub (público o privado con token)
gh repo create cybertiger06/allio-updates --public --description "Allio Client update manifests & instance files"

# Clonar localmente
git clone https://github.com/cybertiger06/allio-updates.git

# Primer sync
python sync_to_updates_repo.py --instance betatest --instance permadeath
cd allio-updates && git add -A && git commit -m "initial" && git push
```

## Límites de GitHub

- **Archivos individuales**: máx 100 MB (los JARs de mods rara vez superan 20 MB)
- **Repo total**: sin límite duro, pero se recomienda < 5 GB
- **Rate limit raw.githubusercontent.com**: ~5000 req/hora (más que suficiente)
- Para archivos > 100 MB, usar Git LFS (ver `.gitattributes`)
