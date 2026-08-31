# Gorlami Open Exam

Ferramenta simples e local para transformar dumps de provas/simulados (texto ou Word) em um formato JSON normalizado e depois estudá-los como uma prova eletrônica, com correção automática ao final.

Sem build, sem dependências, sem servidor — é só um `.html` autocontido que roda no navegador.

## Estrutura do projeto

```
gorlami/
└── questions/                  # saída normalizada, usada pelo app
    ├── exam1.json
    ├── simuladomule.json
    ├── images/
    │   └── simuladomule/        # imagens extraídas da prova original, referenciadas pelo json
    │       ├── image1.jpeg
    │       └── ...
    └── exam-app.html            # o "Open Exam" — app de prova eletrônica
```

- **`questions/*.json`** é o resultado normalizado do material original de cada prova, um arquivo JSON por prova.
- **`questions/images/<nome-da-prova>/`** guarda as imagens que a questão original referenciava (prints de tela, trechos de RAML/DataWeave, diagramas de flow etc.), extraídas do arquivo fonte e referenciadas pelo JSON.
- **`questions/exam-app.html`** é o app que lê um desses JSONs e conduz a prova.

## Formato do JSON

Cada prova normalizada é um array de questões:

```json
[
  {
    "id": 1,
    "question": "Texto da pergunta (pode ter múltiplas linhas)",
    "alternatives": [
      { "label": "A", "text": "..." },
      { "label": "B", "text": "..." },
      { "label": "C", "text": "..." },
      { "label": "D", "text": "..." }
    ],
    "answer": "D",
    "images": ["images/simuladomule/image3.jpeg"],
    "note": "opcional — usado quando a fonte original não trazia o texto da alternativa/gabarito"
  }
]
```

Campos:

| Campo          | Obrigatório | Descrição |
|----------------|-------------|-----------|
| `id`           | sim         | identificador da questão dentro da prova |
| `question`     | sim         | enunciado (`\n` preservado para múltiplos parágrafos) |
| `alternatives` | sim (pode ser `[]`) | lista `{label, text}`; `text` pode ser `""` quando a fonte original só trazia a alternativa numa imagem |
| `answer`       | não         | letra da alternativa correta; `null`/ausente quando a fonte não trazia gabarito legível |
| `images`       | não         | caminhos relativos (a partir de `questions/`) para imagens associadas à questão |
| `note`         | não         | observação sobre alguma limitação da extração (ex.: alternativas visíveis só na imagem) |

O app é tolerante a pequenas variações desse formato (aceita `options` como objeto `{A: "...", B: "..."}` em vez de `alternatives`, por exemplo), mas o formato acima é o padrão gerado.

## Como usar o app de prova (`exam-app.html`)

1. Abra `questions/exam-app.html` no navegador (duplo clique funciona; ele não precisa de servidor).
2. Na tela inicial, selecione (ou arraste) um dos arquivos `.json` de `questions/`.
3. Responda as questões navegando por "Anterior/Próxima" ou clicando nos números da grade — quadradinhos respondidos ficam destacados.
4. Na última questão, clique em **Finalizar prova** para ver:
   - percentual de acerto e contagem de acertos / erros / em branco;
   - revisão questão a questão, com a alternativa correta em verde e a sua marcação em vermelho quando errada;
   - questões sem gabarito na fonte aparecem marcadas como "sem gabarito" e não entram no cálculo do percentual.
5. "Fazer outra prova" volta para a tela inicial e permite carregar outro JSON.

> As imagens são resolvidas por caminho relativo à pasta do HTML (`images/<prova>/...`). Se você mover `exam-app.html` para outro lugar, mova a pasta `images/` junto (ou ajuste os caminhos no JSON). Se o navegador bloquear o carregamento das imagens ao abrir via duplo clique (`file://`), suba um servidor local simples a partir da pasta `questions/`:
> ```bash
> cd questions && python3 -m http.server 8000
> ```
> e acesse `http://localhost:8000/exam-app.html`.

## Adicionando uma nova prova

1. A partir do material original da prova (texto, `.docx` etc.), normalize manualmente (ou com um script) para `questions/minhaProva.json`, seguindo o formato acima.
2. Se houver imagens/exhibits, extraia-as para `questions/images/minhaProva/` e referencie os caminhos relativos no campo `images` de cada questão.
3. Abra `exam-app.html` e selecione o novo JSON — nenhuma alteração de código é necessária.

## Limitações conhecidas

- Quando a fonte original trazia as alternativas apenas como imagem (sem texto extraível), `alternatives` fica com `text: ""` ou vazio e um `note` explica o motivo — a imagem continua referenciada em `images` para consulta visual.
- Quando não foi possível identificar a resposta correta na fonte, `answer` fica `null` e a questão não conta na pontuação final (aparece como "sem gabarito").
