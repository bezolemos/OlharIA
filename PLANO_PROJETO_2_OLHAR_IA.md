# Projeto 2 — OLHAR-IA: plano de desenvolvimento

> Guia de trabalho para Bernardo e mais um integrante. Linguagem: **Python**. Atualizem os nomes e as datas quando definirem o calendário com o professor.

## 1. O que vamos construir

Um programa que recebe imagens de **uma webcam ou de um vídeo de teste autorizado**, detecta pessoas, mostra quantas estão visíveis **naquele instante** e compara esse número com a capacidade configurada do espaço. O contexto de demonstração será um laboratório, sala ou evento acadêmico. A ideia é ajudar a acompanhar a **ocupação do ambiente** para apoiar a organização dos espaços de aprendizagem.

**Entrega mínima (MVP):** abrir a câmera, detectar apenas pessoas, exibir a contagem atual na tela, mostrar o limite configurado e encerrar sem deixar a câmera ocupada. Após validar essa parte, adicionar alerta e histórico de contagens agregadas.

**Limites importantes:** contar pessoas em um quadro **não** equivale a medir presença individual, identificar alunos, avaliar atenção/engajamento ou saber quantas pessoas diferentes passaram pelo local. Alguém oculto pode não ser detectado e alguém no fundo pode entrar na contagem. A relação entre ocupação e aprendizagem é uma **hipótese de uso a investigar**, não uma conclusão gerada pela câmera.

## 2. Fluxo do protótipo

```text
Webcam ou vídeo autorizado → quadro → detector de pessoas → contagem atual
                                                      ↓
                       limite do espaço → taxa de ocupação e alerta
                                                      ↓
                                 tela local + CSV só com números agregados
```

- Começar com **uma câmera e um ambiente**. O operador informa o nome do espaço e sua capacidade; o programa não deduz capacidade pela imagem.
- Para testar sem webcam, permitir selecionar um arquivo de vídeo local. Vídeos usados nos testes só devem entrar no repositório se houver autorização apropriada; a preferência é não publicá-los.
- Usar inicialmente um modelo de detecção de objetos já treinado, filtrando a classe `person`. **Não é necessário treinar uma IA do zero.**
- Desenhar caixas sobre as pessoas apenas na visualização local da demonstração. Não salvar nem transmitir quadros, rostos ou áudio. O CSV registra somente horário, espaço, contagem, capacidade e estado; a interface de vídeo ainda exibe pessoas e exige cuidado no uso.
- Alertas do MVP são **visuais e locais**. Não há envio automático de dados a alunos, professores ou sistemas da faculdade.

## 3. Ferramentas e bibliotecas

**3 pacotes externos diretos** no `requirements.txt`:

| Pacote | Para quê | Onde será usado |
| --- | --- | --- |
| `opencv-python` (`cv2`) | Abrir webcam/vídeo e desenhar texto/caixas na janela | `camera.py`, `app.py` |
| `ultralytics` | Carregar modelo YOLO pré-treinado e obter detecções da classe pessoa | `detector.py` |
| `pytest` | Rodar testes de regras sem depender de câmera | `tests/test_ocupacao.py` |

Módulos **já incluídos no Python**, portanto sem instalação separada: `csv`, `datetime`, `pathlib` e `argparse` (e `time`, se for necessário controlar o intervalo do registro). Pacotes instalados indiretamente pelos três acima não estão contados como bibliotecas escolhidas pela equipe. Fixem versões compatíveis e testadas no `requirements.txt` ao configurar os dois computadores; usem a **mesma versão principal do Python**, a combinar, em ambos.

O modelo pré-treinado pode precisar de download na primeira execução; anotar no README qual peso e qual versão foram usados. Não commitar pesos nem vídeos grandes. Antes de qualquer distribuição ou uso institucional, verificar a licença vigente do modelo/pacote.

## 4. Arquivos planejados

**10 arquivos versionados inicialmente: 6 arquivos Python + 4 de apoio.** `dados/ocupacao.csv`, o ambiente virtual e os pesos baixados são gerados localmente e ficam fora do Git.

```text
projeto-2-olhar-ia/
├── app.py                     # Bernardo: liga os módulos e apresenta o resultado
├── camera.py                  # Colega: lê e libera webcam ou vídeo
├── detector.py                # Bernardo: identifica pessoas no quadro
├── ocupacao.py                # Colega: calcula percentual e estado/alerta
├── historico.py               # Colega: grava apenas contagens agregadas em CSV
├── tests/
│   └── test_ocupacao.py       # Colega: testa capacidade e limites
├── README.md                  # Bernardo: instalação, execução e limitações
├── PLANO_PROJETO_2_OLHAR_IA.md # Este guia: ambos atualizam o progresso
├── requirements.txt           # Bernardo: dependências verificadas pela dupla
└── .gitignore                 # Bernardo: arquivos locais que não entram no Git
```

Conforme o programa crescer, novos arquivos podem ser criados. A contagem acima descreve a **estrutura inicial**, não um limite artificial. Adicionar ao `.gitignore` pelo menos `.venv/`, `__pycache__/`, `*.pyc`, `dados/`, `*.pt` e arquivos de vídeo usados localmente.

## 5. Contrato entre os módulos

Definir estas entradas e saídas **antes de programar separadamente**:

| Módulo | Entrada | Saída esperada |
| --- | --- | --- |
| `camera.py` | `fonte`: índice de webcam, como `0`, ou caminho de vídeo | Um quadro por vez; sinal claro de fim/erro; recurso liberado ao encerrar |
| `detector.py` | Quadro da câmera | Lista de pessoas detectadas; para cada uma, `(x1, y1, x2, y2, confianca)`; coordenadas em pixels |
| `ocupacao.py` | `quantidade` detectada e `capacidade` positiva | Percentual e estado (`normal`, `atenção`, `lotado`); exemplo: atenção em **80%**, lotado em **100%** ou mais |
| `historico.py` | Horário, ambiente, quantidade, capacidade, estado | Uma linha em CSV no máximo a cada **10 segundos**; nenhum quadro ou identificador pessoal |
| `app.py` | `--fonte`, `--ambiente` e `--capacidade` | Janela com contagem, percentual e estado; fechamento com tecla `q` |

**Regra do percentual:** `quantidade / capacidade × 100`. Por exemplo, 8 pessoas para capacidade 10 → 80% → `atenção`. Se `capacidade <= 0`, mostrar erro de configuração. Um quadro sem pessoas → contagem 0. O percentual acima de 100% deve ser exibido, pois indica que o limite foi ultrapassado.

**Atenção ao detector:** conferir no modelo escolhido qual identificador corresponde a `person`; não assumir que todo objeto detectado é uma pessoa. Configurar um limiar inicial de confiança (por exemplo, `0,5`) e calibrá-lo com testes. Uma pessoa visível deve gerar **uma** detecção após o pós-processamento do modelo, mas isso deve ser medido; duplicatas e oclusões podem causar erro.

## 6. Quem faz o quê

| Responsável | Principal responsabilidade | Entrega observável no GitHub |
| --- | --- | --- |
| **Bernardo** | Definir estrutura e dependências; estudar saída do YOLO; implementar `detector.py` e `app.py`; integrar, medir erros e escrever o README | Commits próprios com demonstração de uma pessoa, várias pessoas e integração |
| **Colega** | Implementar `camera.py`, `ocupacao.py` e `historico.py`; criar `tests/test_ocupacao.py`; documentar decisões e executar testes em outro computador | Commits próprios com leitura da câmera, regras de capacidade, CSV e testes |
| **Ambos** | Escolher vídeos/cenários autorizados; testar juntos; revisar código um do outro; registrar limitações e atualizar este cronograma | Revisões/PRs e commits de cada participante |

**Como trabalhar sem atropelar arquivos:** Bernardo cria a estrutura e documenta os contratos; cada pessoa trabalha em uma branch própria (`feat/detector-app` e `feat/camera-ocupacao`). O colega pode implementar a câmera usando uma saída simulada do detector. Bernardo pode integrar usando um quadro de teste. Fazer PRs pequenos para a branch principal, revisar os contratos e reunir os módulos até o fim da semana 3. Cada integrante faz seus próprios commits e é listado nos créditos do README; o histórico do Git mostra as contribuições reais.

## 7. Cronograma sugerido — 5 semanas

> Cada semana começa quando vocês iniciarem oficialmente. Ajustem as datas se o professor definir um prazo menor. **Fim da semana 3 é o marco mínimo demonstrável.**

| Semana | Bernardo | Colega | Critério de entrega conjunto |
| --- | --- | --- | --- |
| **1 — definição e preparação** | Criar repositório, este plano, `requirements.txt`, `.gitignore`; testar uma inferência em imagem local; definir modelo e limiar iniciais | Testar abertura e encerramento da webcam em `camera.py`; conferir leitura de vídeo local; estudar `VideoCapture` e os contratos | Ambos instalam as dependências, o vídeo abre, o modelo detecta pessoa numa imagem; decisões registradas no README |
| **2 — dois módulos em paralelo** | Escrever `detector.py`: filtrar pessoas e retornar caixas + confiança; testar 0, 1 e várias pessoas | Escrever `ocupacao.py` e testes: 0%, 79%, 80%, 100%, >100% e capacidade inválida; finalizar `camera.py` | Cada parte roda separadamente e respeita as entradas/saídas da seção 5; cada um faz seus commits |
| **3 — integração e MVP** | Criar `app.py`: usar a câmera, chamar detector e ocupação, mostrar quantidade/alerta e tecla `q`; corrigir integração | Revisar o fluxo da câmera, tratar fim de vídeo e falta de webcam; revisar PR do Bernardo e repetir os testes em outro PC | Demonstração ao vivo ou em vídeo autorizado: quadro → detecções → contagem → alerta; câmera liberada ao encerrar |
| **4 — histórico e validação** | Medir contagem contra observação manual em cenários distintos; documentar falsos positivos, pessoas encobertas e desempenho | Criar `historico.py`: CSV agregado, cabeçalho único e uma linha no intervalo definido; integrar com Bernardo; validar que nenhum quadro é salvo | CSV de exemplo gerado localmente, testes passando, tabela de erros em pelo menos 3 cenários: vazio, poucas pessoas, grupo |
| **5 — apresentação e revisão** | Concluir README, instruções de execução e demonstração; alinhar o pitch ao que efetivamente funciona | Revisar instruções em instalação limpa, explicar sua contribuição técnica e ajudar na demonstração | Repositório reproduzível, contribuições dos dois visíveis, vídeo/demo funcionando e limitações descritas honestamente |

### Checklist de entrega

- [ ] **S1:** câmera e modelo testados; contrato validado pelos dois.
- [ ] **S2:** módulos individuais e testes de ocupação prontos.
- [ ] **S3:** MVP integrado e demonstrável.
- [ ] **S4:** histórico agregado e avaliação manual documentada.
- [ ] **S5:** README, créditos, revisão final e apresentação.

## 8. Como verificar se funciona

1. **Teste de regras sem câmera:** executar `pytest`; conferir capacidade inválida, limites de 80% e 100% e contagem zero.
2. **Teste da câmera:** abrir, fechar com `q`, rodar de novo. Testar câmera inexistente e fim de vídeo sem travar.
3. **Teste de contagem:** comparar em pelo menos três cenas com a contagem feita por uma pessoa. Registrar `real`, `detectado`, `erro absoluto = |detectado − real|`, iluminação, posição da câmera e oclusões. Não prometer precisão antes de medi-la.
4. **Teste de privacidade:** verificar que CSV e logs não contêm nomes, imagens, vídeo ou IDs de rastreamento; retirar vídeo/frames usados nos testes do Git.
5. **Teste de reprodução:** o colega segue somente o README para instalar e rodar o protótipo em seu computador.

## 9. Privacidade, autorização e linguagem do pitch

Usar webcam própria ou gravação autorizada para a demonstração. Antes de ligar a qualquer câmera institucional, obter autorização da instituição e definir finalidade, acesso e retenção com o responsável pela proteção de dados. Mesmo sem reconhecimento facial, **quadros de vídeo podem conter dados pessoais**. Evitar expressões como “100% LGPD”, “anonimato garantido”, “custo zero” ou “detecta engajamento”: essas conclusões não decorrem apenas do código. A proposta busca **minimizar dados** ao registrar só números, mas conformidade legal exige análise do uso real.

Para explicar ao professor: **“Nosso protótipo em Python estima, pela imagem de uma câmera, quantas pessoas estão visíveis em um espaço de aprendizagem e avisa quando a ocupação se aproxima da capacidade definida. Testaremos erros e registraremos somente contagens agregadas. A intenção é apoiar a gestão de salas e laboratórios; efeitos sobre atenção e aprendizagem ainda precisariam ser estudados.”**

## 10. Evoluções posteriores (fora da entrega mínima)

- **Dashboard/gráficos** com dados agregados; só adicionar biblioteca de gráficos quando isso entrar no escopo.
- **Mapa de calor espacial** exige definir método, câmera fixa e calibração; não confundir com gráfico de horários de maior ocupação.
- **Contagem de entradas e saídas** requer rastreamento e uma linha virtual, com cuidados adicionais; a contagem de cada quadro não faz isso.
- **Previsão de picos, recomendação de espaços, integração com sistemas da Ânima e múltiplas câmeras** exigem dados, autorização, infraestrutura e testes próprios.

## Referências técnicas e de privacidade

- [OpenCV: `VideoCapture`](https://docs.opencv.org/4.13.0/d8/dfe/classcv_1_1VideoCapture.html) — abrir, ler e liberar fonte de vídeo.
- [Ultralytics: predição](https://docs.ultralytics.com/modes/predict/) — filtro de classes e inferência com modelo pré-treinado.
- [Ultralytics: API Python](https://docs.ultralytics.com/usage/python/) — resultados, integração e execução em Python.
- [Python: módulo `csv`](https://docs.python.org/3/library/csv.html) — escrita do histórico agregado.
- [Portal Gov.br: tratamento de dados pessoais e LGPD](https://www.gov.br/planejamento/pt-br/acesso-a-informacao/tratamento-de-dados-pessoais) — definição e responsabilidades no tratamento.

**Base do escopo:** resumo “Projeto Câmera IA” e materiais do pitch “OLHAR-IA” compartilhados com a equipe. Este arquivo organiza uma proposta de execução; prazos e responsabilidades podem ser ajustados pelos dois integrantes.
