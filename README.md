# Odysseus Skill Importer

Ferramenta de terminal (TUI) para importar **skills** de repositórios GitHub para a instalação local do Odysseus. Construída com [Textual](https://textual.textualize.io/).

## Por que usar

Importar skills manualmente — clonar repositório, localizar pastas, copiar arquivos — é repetitivo e propenso a erro. Este projeto automatiza o fluxo: você informa a pasta raiz do Odysseus, adiciona URLs do GitHub, escolhe quais skills importar e acompanha o progresso em tempo real, tudo dentro do terminal.

## Funcionalidades

- **Destino seguro** — skills vão sempre para `<raiz>/data/skills/imported`. A pasta `general` (skills criadas manualmente no Odysseus) nunca é alterada.
- **Validação da instalação** — o botão **Verify** confirma que a pasta é um Odysseus válido (precisa conter `data/skills`). Caminhos apontando para dentro da instalação (ex.: `...\data\skills`) são normalizados automaticamente até a raiz.
- **Caminho lembrado** — a pasta raiz do Odysseus é salva entre execuções e pré-preenchida na próxima abertura.
- **Vários repositórios** — adicione quantas URLs GitHub quiser antes de escanear.
- **Clone rápido** — repositórios são clonados com `git clone --depth 1`.
- **Detecção inteligente de skills** — suporte a manifests `SKILL.md` (nome/descrição via YAML front matter) e fallback para layouts `skills/*/README.md`.
- **Seleção interativa** — lista com checkbox, nome e descrição resumida por skill.
- **Confirmação antes de gravar** — diálogo modal evita importações acidentais.
- **Status em tempo real** — cada skill passa por `Pending → Importing → Success/Failed`.
- **Sem sobrescrita** — skills que já existem no destino são reportadas como falha, em vez de serem substituídas.

## Requisitos

| Requisito | Versão / observação |
|-----------|---------------------|
| Python | 3.11 ou superior |
| Git | Disponível no `PATH` (`git --version`) |

## Instalação

Clone o repositório e crie um ambiente virtual:

```bash
git clone https://github.com/s3xz/autoskills.git
cd autoskills
python -m venv .venv
```

**Windows (CMD ou PowerShell):**

```cmd
.\.venv\Scripts\activate.bat
pip install -r requirements.txt
```

**Linux / macOS:**

```bash
source .venv/bin/activate
pip install -r requirements.txt
```

### Instalação como pacote (opcional)

```bash
pip install -e .
```

Isso registra o comando `odysseus-importer` no ambiente virtual.

## Como usar

### Iniciar a TUI

```bash
python main.py
```

Ou, se instalou com `pip install -e .`:

```bash
odysseus-importer
```

### Fluxo passo a passo

1. Informe a **pasta raiz do Odysseus** (ex.: `C:\odysseus`) e pressione **Verify**.
   - Em caso de sucesso, o caminho é salvo e o destino de importação (`<raiz>/data/skills/imported`) é exibido.
2. Cole uma **URL do GitHub** e pressione **Add Repo** (repita para vários repositórios).
3. Pressione **Scan Skills** — os repositórios são clonados e escaneados.
4. Marque as skills desejadas (ou use **Add All**) e pressione **Confirm**.
5. Aprove o diálogo de confirmação e acompanhe o status de cada importação.

### Atalhos de teclado

| Tela | Atalho | Ação |
|------|--------|------|
| Global | `Ctrl+Q` | Sair |
| Setup | `Q` | Sair |
| Skills | `A` | Selecionar todas |
| Skills | `Espaço` | Alternar seleção da skill em foco |
| Skills | `Esc` | Voltar |

## Como as skills são detectadas

O scanner procura skills nesta ordem:

1. **Manifest `SKILL.md`** — qualquer pasta que contenha um `SKILL.md`. Nome e descrição vêm do front matter YAML; se ausentes, usa o nome da pasta e o primeiro parágrafo do arquivo.
2. **Layout convencional** — pastas dentro de `skills/` (ou na raiz do repositório) que possuem `README.md`.

## Onde os arquivos vão parar

```
<raiz-odysseus>/
└── data/
    └── skills/
        ├── general/      ← não modificado por esta ferramenta
        └── imported/     ← destino das skills importadas
            └── minha-skill/
```

## Configuração persistente

O caminho da raiz do Odysseus é armazenado em:

| Sistema | Local |
|---------|-------|
| Windows | `%APPDATA%\odysseus-importer\config.json` |
| Linux / macOS | `$XDG_CONFIG_HOME/odysseus-importer/config.json` ou `~/.config/odysseus-importer/config.json` |

Falhas de leitura/gravação são ignoradas silenciosamente — a ferramenta continua funcionando sem cache.

## Estrutura do projeto

```
main.py                          # ponto de entrada
odysseus_importer/
├── app.py                       # App Textual e ciclo de vida do workdir temporário
├── app.tcss                     # tema visual da TUI
├── config.py                    # cache do caminho do Odysseus entre execuções
├── models.py                    # Skill, ImportStatus, ScanReport
├── scanner.py                   # clone git + detecção de skills
├── importer.py                  # validação da raiz + cópia para data/skills/imported
├── screens/
│   ├── setup.py                 # pasta raiz + URLs de repositório
│   ├── skills.py                # lista selecionável com status ao vivo
│   └── confirm.py               # diálogo modal de confirmação
└── widgets/
    └── skill_row.py             # linha: checkbox + nome/descrição + status
tests/
└── test_core.py                 # testes unitários do scanner e importer
```

## Desenvolvimento

Instale as dependências de desenvolvimento e rode os testes:

```bash
pip install -e ".[dev]"
pytest
```

## Solução de problemas

| Problema | Possível causa |
|----------|----------------|
| `git is not installed or not on PATH` | Instale o Git e reinicie o terminal |
| `does not look like an Odysseus installation` | Informe a pasta raiz do Odysseus, não `data/skills` |
| `'nome-da-skill' already exists` | A skill já está em `imported/` — remova manualmente se quiser reimportar |
| `Timed out cloning` | Repositório grande ou conexão lenta; tente novamente |
| `No skills detected` | O repositório não segue os layouts suportados (`SKILL.md` ou `skills/*/README.md`) |
