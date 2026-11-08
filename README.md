
# Monitorização de Tráfego Rodoviário — PIV

Sistema de visão por computador desenvolvido em Python e OpenCV para detetar, acompanhar e contar veículos numa sequência de vídeo de uma autoestrada. O programa estima também a velocidade dos veículos e monitoriza o fluxo de circulação nas três faixas de rodagem.

O projeto foi desenvolvido no âmbito do **2.º Trabalho Prático da unidade curricular de Processamento de Imagem e Visão (PIV)**, no ISEL, durante o ano letivo de 2025/2026.

## Objetivos

O sistema analisa o vídeo `AutoEstrada.avi`, captado na Autoestrada A2, no sentido Norte–Sul, no troço de acesso a Almada após a Ponte 25 de Abril.

As principais funcionalidades são:

- Detetar veículos em movimento dentro de uma região de interesse (ROI).
- Atribuir um identificador único a cada veículo e acompanhar a sua posição ao longo dos frames.
- Contar os veículos sem contabilizar repetidamente o mesmo veículo.
- Estimar a velocidade de cada veículo em km/h.
- Identificar a faixa de rodagem em que cada veículo circula.
- Calcular a velocidade média dos veículos por faixa.
- Assinalar os veículos cuja velocidade estimada fica abaixo do limite definido para a respetiva faixa.
- Apresentar no vídeo as caixas delimitadoras (*bounding boxes*), os identificadores e as velocidades.

## Tecnologias utilizadas

- **Python**
- **OpenCV (`cv2`)** — leitura e processamento do vídeo, operações morfológicas, extração de contornos e visualização.
- **NumPy** — operações sobre matrizes e cálculo do fundo através da mediana temporal.
- **Math** — cálculo de distâncias euclidianas.

## Funcionamento

O processamento é realizado frame a frame e segue as etapas descritas abaixo.

### 1. Região de interesse e faixas de rodagem

É criada uma máscara poligonal que limita a análise à zona relevante da estrada, excluindo áreas exteriores e tráfego no sentido contrário. A região é dividida em três faixas:

| Faixa | Velocidade mínima de referência |
|---|---:|
| Esquerda | 90 km/h |
| Central | 70 km/h |
| Direita | 50 km/h |

Estes valores são os limiares utilizados pelo programa para assinalar circulação abaixo da velocidade definida para cada faixa.

### 2. Estimação do fundo

Para obter uma referência da imagem sem os veículos em movimento, o programa recolhe uma amostra de **50 frames** e calcula a mediana temporal de cada pixel. A imagem resultante é convertida para escala de cinzentos e utilizada como fundo estático.

### 3. Deteção de movimento

Em cada frame, é calculada a diferença absoluta entre a imagem atual e o fundo estimado. É aplicado um limiar de **35** para produzir uma máscara binária: os pixels com alterações suficientemente significativas são considerados regiões ativas.

### 4. Processamento morfológico

A máscara binária é tratada com operações morfológicas para reduzir ruído e juntar partes fragmentadas dos veículos:

- **Dilatação:** expande as regiões detetadas e ajuda a unir partes próximas.
- **Fecho (*closing*):** preenche pequenos buracos e melhora a coesão das regiões.

De seguida, a máscara da região de interesse é aplicada para remover deteções fora da estrada.

### 5. Extração e filtragem de deteções

O OpenCV identifica os contornos das regiões ativas e calcula uma *bounding box* para cada contorno.

São aplicados dois critérios principais:

- **Área:** são ignoradas regiões com área igual ou inferior a 400 pixels.
- **Proporção largura/altura:** quando a razão é superior a 2.0, o programa tenta separar a região em duas deteções, para lidar com veículos próximos ou parcialmente unidos na imagem.

### 6. Tracking dos veículos

O sistema associa as deteções de frames consecutivos através da distância euclidiana entre os centróides.

- Cada veículo recebe um ID único.
- Uma deteção é associada a um veículo já registado quando a distância entre os centróides não ultrapassa **50 pixels**.
- Se um veículo deixar de ser detetado, o seu registo é mantido durante, no máximo, **3 frames**, permitindo tolerar falhas momentâneas de segmentação.
- As deteções sem correspondência são registadas como novos veículos.

Este mecanismo permite manter a identidade dos veículos ao longo do vídeo e reduzir contagens duplicadas.

### 7. Contagem e estimativa da velocidade

Um veículo é contabilizado quando o seu movimento cruza a linha virtual definida em `y = 120` e ainda não foi contado.

A velocidade é estimada através do deslocamento vertical do centróide desde a posição inicial do veículo, do tempo decorrido entre frames e da taxa de frames por segundo (FPS) do vídeo. O fator de calibração utilizado no código é **6.6 pixels por metro**, ajustado experimentalmente para compensar parcialmente a perspetiva da câmara.

O resultado é convertido para km/h. Para reduzir oscilações, as atualizações seguintes da velocidade utilizam uma média ponderada: 80% do valor anterior e 20% da nova estimativa. O cálculo só é atualizado quando o deslocamento ultrapassa 30 pixels.

> **Nota sobre a calibração:** o relatório refere que a medição inicial das marcas rodoviárias indicava aproximadamente 10,3 px/m, mas que esse valor produzia velocidades demasiado baixas na zona analisada. Por esse motivo, foi utilizado o valor experimental de 6,6 px/m.

### 8. Monitorização por faixa e visualização

A posição do centróide é utilizada para determinar a faixa de rodagem. Para cada faixa, o programa acumula as velocidades dos veículos contabilizados e calcula a respetiva média.

Quando a velocidade estimada de um veículo fica abaixo do limiar da sua faixa, a caixa delimitadora é apresentada a vermelho e a indicação `BAIXA` é incluída na legenda. Caso contrário, a caixa é apresentada a verde.

A janela de visualização apresenta ainda o ID do veículo, a velocidade estimada em km/h, a indicação de velocidade baixa quando aplicável, o total de veículos contabilizados, os limites das três faixas e os pontos de referência da calibração. As velocidades médias acumuladas por faixa são também apresentadas na consola.

## Estrutura do projeto

Uma organização possível para o repositório é:

```text
PIV-Trabalho-2/
├── README.md
├── main.py
└── AutoEstrada.avi
```

- `main.py` — ficheiro Python com o algoritmo de deteção, tracking, contagem e estimativa de velocidade. O nome pode ser ajustado ao nome real do ficheiro do projeto.
- `AutoEstrada.avi` — vídeo utilizado como entrada pelo programa.

O código abre o vídeo através do caminho relativo `AutoEstrada.avi`. Por isso, o vídeo deve estar na mesma pasta em que o programa é executado, ou o caminho no código deve ser atualizado.

## Instalação e execução

### 1. Requisitos

É necessário ter Python instalado. As bibliotecas externas utilizadas são OpenCV e NumPy.

Instala as dependências com:

```bash
pip install opencv-python numpy
```

### 2. Preparar os ficheiros

Coloca o ficheiro Python e o vídeo `AutoEstrada.avi` na mesma pasta.

### 3. Executar

No terminal, abre a pasta do projeto e executa:

```bash
python main.py
```

Se o ficheiro Python tiver outro nome, substitui `main.py` pelo nome correto.

### 4. Controlos

Durante a reprodução, prime **`q`** para terminar a execução. No final, o programa liberta a captura de vídeo e fecha as janelas do OpenCV.

## Resultados

De acordo com os testes descritos no relatório, o vídeo de teste continha **12 veículos**. O algoritmo contabilizou os mesmos 12 veículos, correspondendo a uma precisão de **100% neste cenário específico**.

O relatório apresenta a seguinte evolução das velocidades médias por faixa:

| Frame | Esquerda (km/h) | Central (km/h) | Direita (km/h) |
|---:|---:|---:|---:|
| 8 | 86 | 74 | 69 |
| 44 | 76 | 67 | 69 |
| 149 | 76 | 67 | 61 |
| 204 | 82 | 76 | 61 |
| 234 | 84 | 75 | 61 |

A precisão indicada diz respeito apenas à sequência de vídeo testada e não deve ser interpretada como uma garantia de desempenho em quaisquer condições de tráfego, iluminação ou perspetiva.

## Limitações

- A deteção depende da qualidade da subtração de fundo e pode ser afetada por sombras, reflexos e alterações de iluminação.
- A associação entre frames utiliza a distância entre centróides, pelo que situações de oclusão ou veículos muito próximos podem dificultar o tracking.
- A separação de regiões largas através da proporção largura/altura é uma heurística e pode não funcionar corretamente em todos os casos.
- A conversão de pixels para metros utiliza um fator fixo. Como a perspetiva altera a escala ao longo da imagem, a estimativa de velocidade é aproximada e depende da calibração e da zona analisada.
- Os limiares geométricos e de velocidade foram ajustados ao vídeo e ao local considerados neste trabalho.

## Autor e contexto académico

- **Autor:** Diogo Santiago — 51998
- **Curso:** Licenciatura em Engenharia Informática e Multimédia (LEIM)
- **Instituição:** ISEL
- **Turma:** 52D
- **Unidade curricular:** Processamento de Imagem e Visão (PIV)
- **Trabalho:** 2.º Trabalho Prático
- **Docente:** João Pedro Costa
- **Ano letivo:** 2025/2026
- **Data de entrega:** 07/12/2025

## Referências

- Materiais de apoio e acetatos disponibilizados pelo docente.
- [Documentação oficial do OpenCV](https://docs.opencv.org/)
- [Google Maps](https://www.google.com/maps/)
