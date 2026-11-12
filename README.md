# Monitorização de Tráfego Rodoviário (PIV)

Sistema de visão por computador em **Python e OpenCV** que deteta, segue e conta veículos num vídeo de autoestrada, estima a velocidade de cada um e assinala os que circulam abaixo do limite mínimo da sua faixa.

Segundo trabalho prático de **Processamento de Imagem e Visão** (LEIM, ISEL, 2025/2026). Vídeo de teste: A2, sentido Norte-Sul, acesso a Almada após a Ponte 25 de Abril.

## O que faz

- Deteta veículos em movimento dentro de uma região de interesse (ROI) poligonal.
- Atribui um identificador a cada veículo e segue-o entre frames, sem contar o mesmo veículo duas vezes.
- Estima a velocidade em km/h.
- Identifica a faixa (esquerda, central, direita) e calcula a velocidade média por faixa.
- Desenha as caixas, o ID e a velocidade no vídeo; fica a vermelho com `BAIXA` se estiver abaixo do limite da faixa.

| Faixa | Velocidade mínima de referência |
|---|---|
| Esquerda | 90 km/h |
| Central | 70 km/h |
| Direita | 50 km/h |

## Método

1. **ROI e faixas.** Máscara poligonal que exclui o tráfego em sentido contrário, dividida em três faixas.
2. **Fundo.** Mediana temporal pixel a pixel de 50 frames consecutivos, em escala de cinzentos.
3. **Movimento.** Diferença absoluta entre cada frame e o fundo, com limiar 35.
4. **Morfologia.** Dilatação e fecho para unir as partes de um mesmo veículo (sem isto, o para-brisas dividia o carro em dois).
5. **Deteções.** Contornos com `findContours`, ignorando regiões com área até 400 píxeis.
6. **Tracking.** Associação gananciosa pela menor distância euclidiana entre centróides (máximo de 50 píxeis). Um veículo não detetado mantém-se em memória até 3 frames.
7. **Contagem.** Um veículo conta quando o centróide cruza a linha virtual `y = 120`, e só uma vez.
8. **Velocidade.** Deslocamento vertical desde a primeira deteção, dividido pelo tempo decorrido (a partir dos FPS), convertido com 6,6 píxeis por metro e suavizado (80% valor anterior, 20% nova medida). Só se atualiza após 30 píxeis de deslocamento.

**Calibração.** Medir as marcas rodoviárias (3 m cada) dá cerca de 10,3 px/m perto da câmara, mas isso dava velocidades de uns 30 km/h, porque a perspetiva faz com que um metro real ocupe menos píxeis longe da câmara. O fator foi ajustado experimentalmente para 6,6 px/m.

## Resultados

No vídeo de teste passam **12 veículos** e o sistema contou **12**.

| Frame | Esquerda (km/h) | Central (km/h) | Direita (km/h) |
|---|---|---|---|
| 8 | 86 | 74 | 69 |
| 44 | 76 | 67 | 69 |
| 149 | 76 | 67 | 61 |
| 204 | 82 | 76 | 61 |
| 234 | 84 | 75 | 61 |

Isto vale para **uma única sequência curta com tráfego fluido e boa visibilidade**; não é uma garantia noutras condições. Também não existem velocidades reais medidas para comparar, por isso as velocidades são plausíveis, não validadas.

## Executar

Requisitos: Python 3, `opencv-python` e `numpy`.

```bash
pip install opencv-python numpy
```

Abrir o notebook `TP2_51998_PIV.ipynb` e executar as células, com o vídeo `AutoEstrada.avi` na mesma pasta. Carregar `q` na janela do vídeo para terminar.

## Limitações

- A subtração de fundo é sensível a sombras, reflexos e mudanças de luz.
- O tracking por centróides pode falhar com oclusões ou veículos muito próximos.
- **A separação de regiões largas não funciona como descrito.** O código tenta dividir deteções com largura/altura superior a 2, mas dentro desse caso testa `h > w`, que nunca é verdadeiro (as duas condições são incompatíveis). Na prática, nunca divide nada.
- A conversão píxel-metro é um fator fixo, apesar de a escala variar com a perspetiva.
- A velocidade é uma média desde a primeira deteção, não instantânea.
- Os limiares foram ajustados a este vídeo e a este local.

## Autor

Diogo Santiago (51998), LEIM 52D, ISEL. Docente: João Pedro Costa. Entrega: 07/12/2025. O relatório está em `docs/`.

Referências: materiais da unidade curricular, documentação do OpenCV.
