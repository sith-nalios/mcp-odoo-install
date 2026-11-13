# Installation MCP Server Odoo

Scripts d'installation du serveur MCP `mcp-server-odoo` pour Claude Desktop.

## Windows

1. Appuie sur `Win` + `R`, tape `powershell`, puis `Entrée`
2. Colle cette commande dans la fenêtre PowerShell (pas dans cmd !) :

```powershell
irm https://raw.githubusercontent.com/sith-nalios/mcp-odoo-install/main/install-mcp-odoo-windows.ps1 | iex
```

> **Important :** ne pas télécharger et double-cliquer le fichier `.ps1` — coller la commande dans PowerShell comme indiqué ci-dessus.

### Variante sans droits admin (erreur pywin32 récurrente)

Si l'erreur pywin32 (`os error 32`) revient à chaque redémarrage de Claude Desktop et que l'utilisateur n'a pas les droits admin sur son poste, utiliser ce script à la place : il installe `mcp-server-odoo` une seule fois de façon persistante (`uv tool install`) au lieu de le réinstaller à chaque lancement de Claude Desktop.

```powershell
irm https://raw.githubusercontent.com/sith-nalios/mcp-odoo-install/main/install-mcp-odoo-windows-persistent.ps1 | iex
```

> Pour changer de version de `mcp-server-odoo` plus tard, il faut relancer ce script plutôt que de simplement modifier la config.

## Mac

Ouvre un Terminal et colle :

```bash
curl -LsSf https://raw.githubusercontent.com/sith-nalios/mcp-odoo-install/main/install-mcp-odoo-mac.sh | bash
```

## Installation manuelle (Windows)

Si le script automatique ne fonctionne pas, voici les étapes manuelles.

### 1. Installer uv

Dans PowerShell (pas cmd) :

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

La commande affiche le chemin d'installation, par exemple :
`C:\Users\TonNom\.local\bin`

### 2. Ajouter uv au PATH

Copie-colle le chemin affiché dans la commande suivante (remplace le chemin) :

```powershell
$env:PATH = "C:\Users\TonNom\.local\bin;$env:PATH"
```

### 3. Installer Python 3.12

```powershell
uv python install 3.12
```

### 4. Configurer le MCP manuellement

Ouvre le fichier de config Claude Desktop : dans l'app, **Paramètres > Développeur > Modifier la configuration** (c'est le seul moyen fiable de trouver le bon fichier, son emplacement dépend de la version installée).

Emplacements possibles, par ordre de priorité du script :
- `%LOCALAPPDATA%\Claude-Data\claude_pzs8sxrjxfjjc\claude_desktop_config.json` (versions récentes)
- `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\claude_desktop_config.json` (Microsoft Store)
- `%APPDATA%\Claude\claude_desktop_config.json` (installeur classique)

Ajoute le bloc suivant dans la section `mcpServers` (en remplaçant les valeurs) :

```json
"mcpServers": {
  "nalios": {
    "command": "C:\\Users\\TonNom\\.local\\bin\\uvx.exe",
    "args": [
      "mcp-server-odoo@0.9.0"
    ],
    "env": {
      "ODOO_URL": "https://ton-instance.odoo.com",
      "ODOO_DB": "nom-de-la-base",
      "ODOO_USER": "ton@email.com",
      "ODOO_API_KEY": "ta_cle_api",
      "ODOO_YOLO": "true",
      "ODOO_MCP_DEFAULT_LIMIT": "100",
      "ODOO_MCP_MAX_LIMIT": "1000"
    }
  }
}
```

> Le chemin vers `uvx.exe` est celui affiché à l'étape 1, avec `\uvx.exe` ajouté à la fin. Les backslashes doivent être doublés (`\\`) dans le JSON.

---

## Ce que fait le script

- Installe `uv`/`uvx` si nécessaire
- Demande l'URL Odoo, la base de données, l'utilisateur et la clé API
- Détecte le `claude_desktop_config.json` réellement utilisé par Claude Desktop (cf. emplacements ci-dessus), en fait une sauvegarde `.bak`, puis y ajoute le serveur MCP sans toucher aux autres
- Affiche à la fin le fichier modifié et la liste des serveurs MCP qu'il contient

Après l'installation, redémarrer Claude Desktop.

## Si le MCP n'apparaît pas dans Claude

1. Compare le fichier affiché par le script (`Claude Desktop mis a jour : ...`) avec celui ouvert par **Paramètres > Développeur > Modifier la configuration**. S'ils diffèrent, signale le chemin réel à l'équipe.
2. Si le script indique que le JSON existant n'est pas valide, corrige-le (la sauvegarde est dans `claude_desktop_config.json.bak`) puis relance.
3. Redémarre complètement Claude Desktop (le quitter depuis la zone de notification, pas seulement fermer la fenêtre).
