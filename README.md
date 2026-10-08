# Yara Box, versão de campo

Documento técnico da Yara Box (Wara Tech), versão de campo: controle, firmware, telemetria e lógica de automação.

**Versão completa, com diagramas:** [index.html](index.html)

> **Sobre o manual antigo.** O manual público em [yarabox.com.br/manual](https://yarabox.com.br/manual) descreve a versão compacta anterior da Yara (540 L/h, bomba 12 V). Ele ajuda a entender a lógica de placa e de firmware, mas não descreve a máquina de campo.

## Por onde começar no GitHub

Os repositórios públicos estão em [github.com/wara-tech](https://github.com/wara-tech). Sugerimos esta ordem:

1. [Yara_Firmware-publico, branch campo-1.0.2](https://github.com/wara-tech/Yara_Firmware-publico/tree/campo-1.0.2): código da versão 1.0.2.
2. [YB-projeto-eletrico-publico](https://github.com/wara-tech/YB-projeto-eletrico-publico): projeto elétrico.
3. [YB-dashboard-thingsboard-publico](https://github.com/wara-tech/YB-dashboard-thingsboard-publico): painel no ThingsBoard.

## Hidráulica

*Informado pelo dono do produto.*

- Duas linhas em paralelo, cada uma com um pré-filtro e uma ultrafiltração (UF). A água entra por uma bomba. As duas linhas se juntam e vão direto ao ponto de consumo (bebedouro com boia), sem reservatório intermediário.
- Na última montagem, uma das linhas trocou o filtro de mídia por um filtro de disco.
- Rede elétrica de 220 V no local de instalação.
- A dosadora de cloro e o medidor de parâmetros da água estão instalados, mas nunca foram testados em campo.

| Válvula | Função |
|---:|---|
| 0 | Saída para o ponto de consumo |
| 2 | Saída da UF1 |
| 3 | Bypass |
| 4 | Esgoto da UF1 |
| 5 | Entrada da mídia 1 |
| 6 | Entrada da mídia 2 |
| 7 | Esgoto da UF2 |
| 8 | Interliga as duas linhas depois das mídias |
| 9 | Saída da UF2 |
| 11 | Retorno do concentrado das UFs para a linha |
| 13 | Esgoto da mídia 1 |
| 15 | Esgoto da mídia 2 |

## Controle

*Lido no código da 1.0.2 e da main.*

- Controlador ESP32-S3.
- MCP23017 com 16 saídas, que no código comandam as válvulas; a saída 12 é o relé da bomba.
- PCF8574 com 3 saídas de solenoide.
- Dois ADS1115 que, juntos, leem 5 canais de pressão, a tensão da bateria e a tensão da bomba.
- Barramento I²C (pelo datasheet, esses chips só têm I²C).
- Telemetria por MQTT para o ThingsBoard.

## Lógica de automação

*Lido no código.*

- A bomba só liga se as válvulas abertas formarem uma das combinações cadastradas.
- Proteção de pressão: a bomba desliga na hora quando qualquer sensor passa do limite configurável `estado_muito_alto`, que fica gravado na placa. Depois do corte, o firmware reabre as válvulas, espera 15 s e confere de novo com 3 bar fixo; se a pressão continuar alta, encerra o tratamento.
- Não existe religamento automático da bomba. O tratamento volta pelo botão ou por comando remoto.
- Retrolavagem (versão 1.0.2): dispara quando o diferencial de pressão passa de 1 bar, e só é checada com a máquina parada.

## Telemetria e versão

- Cerca de 116 mil pontos por dia e 338 mensagens por hora (medido).
- Em 29/09/2026, a máquina rodava a 1.0.2. O código dessa versão está na branch `campo-1.0.2`.

## Onde queremos ajuda

1. **Como fazer a bomba religar sozinha com segurança?** Hoje não existe religamento automático; o tratamento só volta pelo botão ou por comando remoto.
2. **O que fazer depois que a proteção de pressão encerra o tratamento?** Quando a pressão continua alta na segunda checagem, o firmware encerra o tratamento, e a máquina só volta com ação manual ou remota.
3. **Como colocar a dosadora de cloro e o medidor de parâmetros em operação?** Os dois estão instalados, mas nunca foram testados em campo.

## Fotos

As fotos ficam em [`fotos/`](fotos/): tela de operação no ThingsBoard, Yara montada, montagem em campo e painel de controle.
