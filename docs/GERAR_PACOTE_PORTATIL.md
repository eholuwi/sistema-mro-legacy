# Como gerar o pacote portátil do Sistema MRO (manual passo a passo)

Este manual é para você (Luis) rodar sozinho, sem precisar pedir para o Claude. Gerar o
pacote portátil é o comando de sempre — não precisa avisar a TI (não abre porta, só baixa
arquivos do próprio python.org na primeira vez).

## O que é o "pacote portátil"

Um `.zip` com o Sistema MRO **completo e pronto pra rodar** — código + Python + todas as
bibliotecas já instaladas. Quem recebe só extrai, arrasta um atalho pra área de trabalho e
dá dois cliques. Não precisa instalar Python, não precisa rodar `pip install`, nada.

Serve para:
- Trocar o PC-servidor (o atual quebrou, foi trocado, etc.)
- Levar o sistema para testar em outra máquina
- Ter um backup "pronto pra usar" fora do PC-servidor

Diferente do `scripts/release.py` (que gera um zip só com o *código*, e espera que a
máquina destino já tenha o runtime Python montado à mão).

## Passo a passo

1. Abra o PowerShell **nesta pasta** (`sistema-mro`).

2. Rode:

   ```powershell
   venv\Scripts\python.exe scripts\portatil.py
   ```

3. Espere. Na primeira vez demora alguns minutos (baixa o Python embeddable e instala as
   dependências — é o passo lento). Nas próximas vezes, se quiser pular esse download e
   reaproveitar o runtime já montado:

   ```powershell
   venv\Scripts\python.exe scripts\portatil.py --pular-deps
   ```

   Use `--pular-deps` sempre que só mudou **código** (não mudou `requirements.txt`).

4. No fim aparece uma linha assim:

   ```
   Pacote gerado: dist\mro-portatil-6.10.0.zip  (148 MB)
   ```

   Pronto — é esse `.zip` que você leva/distribui.

## Como instalar o pacote na máquina destino

1. Extraia o `.zip` em `C:\MRO` (caminho local e curto — **nunca** dentro de
   OneDrive/Dropbox/Google Drive: o sincronizador trava o banco e pode corromper).

2. Arraste o `MRO.lnk` (já vem dentro da pasta) para a área de trabalho.
   - Se você extraiu em outro lugar que não `C:\MRO`, o atalho pronto não serve — em vez
     disso, clique com botão direito em `criar_atalho.ps1` → "Executar com o PowerShell".
     Ele recria o atalho apontando pro lugar certo.

3. Dois cliques no atalho **MRO**. O navegador abre sozinho em `http://localhost:8501`.
   Outros PCs da rede acessam pelo IP que aparece na janela preta.

4. (Opcional) Para o sistema subir sozinho quando o PC liga: botão direito em
   `instalar_servidor.ps1` → "Executar com o PowerShell". Pede admin, cria a tarefa
   agendada e libera a porta 8501 no firewall.

O banco fica em `dados\mro.db`, backups em `dados\backups\` — essa pasta sobrevive a
atualizações futuras.

## Erros comuns

| Sintoma | Causa |
|---|---|
| `deploy/<arquivo> ausente` | Algum arquivo de `deploy/` foi apagado/renomeado — confira `RAIZ_DO_PACOTE` em `scripts/portatil.py` |
| Script recusa rodar ("rode numa máquina Windows") | O build só funciona em Windows — o embeddable é `-amd64` do Windows |
| `O CI valida Python X.Y mas este interpretador é Z.W` | A versão do Python do seu `venv` não bate com a do `.github/workflows/verify.yml`. Recrie o venv com a versão certa antes de rodar o build |
| Pacote gerado mas o app não sobe na máquina destino | Rode manualmente `runtime\python.exe -s -c "import streamlit"` — se reclamar de módulo faltando, o runtime saiu incompleto; refaça o build sem `--pular-deps` |

## Onde isso está documentado no código (se quiser entender por dentro)

- `scripts/portatil.py` — o script que monta tudo (tem comentários explicando cada
  armadilha já paga, tipo por que não existe mais `MRO.exe`)
- `deploy/iniciar_mro.bat` — o que sobe o sistema na máquina destino
- `deploy/criar_atalho.ps1` — recria o atalho se o pacote foi extraído em outro lugar
- `CLAUDE.md` (raiz do projeto) — tabela "Distribuição (pacote portátil / MRO.lnk)"
