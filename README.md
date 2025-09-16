# EA Estratégia 80/20 (Oitenta_Vinte)

EA de trading algorítmico baseado em confluência 80/20 com filtros técnicos. Prepara sinais robustos para diversos mercados (Forex, Ações, Índices, Commodities, Cripto, B3) e executa gestão de risco/posição de forma automática.

- Estratégia: modelo 80/20 (probabilidades+confiança) + relação ao POC (Volume Profile) + pressão de compra/venda + regime de mercado.
- Filtros: MAMM (3 camadas), Enhanced MFI, ADXW Cloud, OBV MACD, Pullback Detector (opcional), Filtro Pareto (ATR).
- Execução: cálculo consistente de SL/TP, respeito a StopsLevel/Freeze, garantia de R:R mínimo e Volume por Risco (opcional).
- Gestão: Break Even e Trailing Stop baseados em BARRAS (não em ticks) e não regressivos; fechamento opcional na mudança de sinal.
- Diagnóstico: logs detalhados com resumos de sinal/indicadores e métricas de SL/TP/risco.

## 1. Estratégia 80/20 (núcleo de sinal)
O EA calcula:
- ProbAlta/ProbBaixa e Confiança do modelo 80/20.
- Posição do preço em relação ao POC e “pressões” de compra/venda (orderflow simplificado).
- Regime de Mercado (Alta/Baixa/Lateral) para contextualizar.

Pré-sinais:
- Compra: ProbAlta > ThresholdConfluencia, Confiança > ThresholdConfiancaMinima e (Close > POC ou PressãoCompra).
- Venda: ProbBaixa > ThresholdConfluencia, Confiança > ThresholdConfiancaMinima e (Close < POC ou PressãoVenda).

Em seguida, aplica filtros técnicos (abaixo) e uma votação por proporção (PorcentagemIndicadoresMinima). Há ModoFacilitado que relaxa alguns critérios.

## 2. Indicadores e Filtros
- MAMM (3 camadas): avalia tendência/força por cores (VERDE/VERMELHO) e condições “acima/abaixo” e “rompimento” das MAMMs.
- Enhanced MFI: cores (VERDE/VERMELHO/AMARELO/LATERAL) com zonas ampliadas para evitar ruído.
- ADXW Cloud: exige DI+>DI- (compra) ou DI->DI+ (venda) e ADX >= InpADXRLevel para mercado com tendência válida.
- OBV MACD: confirma direção (MACD vs Signal e cor do histograma) e bloqueia sinais quando divergente do lado proposto.
- Pullback Detector (opcional): classifica o pullback (RASO/PROFUNDO/PLANO/COMPLEXO) e pode exigir “operar apenas em pullbacks”.
- Filtro Pareto (ATR): atrNow/atrAvg20 >= multiplicador; corta sinais em baixa volatilidade.

## 3. Lógica de compra e venda (resumo operacional)
Compra:
1) Pré-sinal: ProbAlta/Confiança/POC/Pressões OK.
2) Gate OBV MACD não bloqueia (sem cross contrário e sem histograma contra).
3) Volatilidade OK (Filtro Pareto).
4) Validação por indicadores: MAMM, MFI, ADXWCloud, OBV MACD (votação por proporção).
5) Opcional: se “OperarApenasPullbacks”, exige pullback de baixa classificado.
6) Se aprovado, gera sinal de COMPRA e abre posição.

Venda:
1) Pré-sinal: ProbBaixa/Confiança/POC/Pressões OK.
2) Gate OBV MACD não bloqueia (sem cross contrário e sem histograma contra).
3) Volatilidade OK (Filtro Pareto).
4) Validação por indicadores: MAMM, MFI, ADXWCloud, OBV MACD (votação por proporção).
5) Opcional: se “OperarApenasPullbacks”, exige pullback de alta classificado.
6) Se aprovado, gera sinal de VENDA e abre posição.

## 4. SL/TP, R:R e execução de ordens
- Cálculo base: StopLoss/TakeProfit em pontos a partir do preço de entrada do símbolo.
- Direção garantida: BUY tem SL<entrada e TP>entrada; SELL tem SL>entrada e TP<entrada.
- Respeita StopsLevel e Spread: “EnsureValidStops” ajusta distâncias para o mínimo aceito pelo broker e normaliza preços.
- R:R mínimo: ajusta TP para garantir MinRRRatio vs. distância real do SL.
- FreezeLevel: se necessário, aplica SL/TP pós-abertura (“TryPostSetStops”) fora da zona de freeze.
- Envio com fallback: caso falhe com SL/TP (regras do broker), o EA abre sem stops e ajusta em seguida.
- Volume:
  - Fixo: usa “Volume” de input, normalizado pelo símbolo.
  - Por Risco (UseRiskManagement=true): calcula lote pelo risco em % (RiskPerTradePercent) usando distância do SL e TickValue/TickSize; normaliza por step e min/max do símbolo.

Logs úteis:
- “SL/TP (req->adj) pts”: mostra pontos solicitados vs. pontos enviados após ajustes.
- “R:R ajustado”: evidencia ajuste de TP para R:R mínimo.
- “Volume final”: mostra volume final aplicado (fixo ou por risco).

## 5. Gestão da posição
- Break Even:
  - Contagem por BARRAS (AtrasoBreakEven).
  - Dispara no gatilho (GatilhoBreakEven em pontos positivos) e coloca SL acima/abaixo do preço de abertura com OffsetBreakEven.
  - Respeita mínima distância (StopsLevel/Spread) e Freeze; não regrede SL.

- Trailing Stop:
  - Contagem por BARRAS (AtrasoTrailingStop).
  - Puxa SL pela distância (ValorTrailingStop), respeitando PassoTrailingStop e mínima distância (StopsLevel/Spread).
  - Não regrede SL e respeita FreezeLevel.

- FecharNaMudancaSinal: fecha posição quando o sinal do modelo muda de lado.

## 6. Filtros de horário e dias
- Janela diária por HoraInicio/HoraFim (inclui janela que cruza a meia-noite e “24h” quando iguais).
- Dias da semana ativáveis individualmente.

## 7. Parâmetros principais (visão geral)
- Geral: MagicNumber, Volume, Slippage, ModoDebug, ModoFacilitado, Intervalos de verificação e reentrada.
- Estratégia 80/20: ThresholdConfluencia, ThresholdConfiancaMinima, MinRRRatio, FiltroVolatilidade, SinaisCompra/VendaAtivados.
- Risk Management: UseRiskManagement, RiskPerTradePercent, RiskUseEquity, MinSLPointsForRisk.
- Módulos: VolumeProfile, Estrutura de Mercado, Fibonacci, Volatilidade, Orderflow, Correlação.
- Operações: TakeProfit, StopLoss, FecharNaMudancaSinal, MostrarLinhas.
- ADXW Cloud: Input_AtivarADXWCloud, InpADXPeriod, InpADXRLevel, cores e transparência.
- Enhanced MFI: PeriodoMFI, zonas laterais, volume e sinais.
- OBV MACD: períodos, suavização, volume aplicado, exibição.
- MAMM: períodos/TFs/constantes/divergência, painel e cores.
- Break Even: UsarBreakEven, GatilhoBreakEven, OffsetBreakEven, AtrasoBreakEven (barras).
- Trailing Stop: UsarTrailingStop, ValorTrailingStop, PassoTrailingStop, AtrasoTrailingStop (barras).
- Pullbacks: ativação, lookback e níveis; OperarApenasPullbacks; PeriodoATR.

## 8. Diagnóstico e logs
- ModoDebug habilita:
  - LogSignalConditions: imprime probabilidades, confiança, POC, pressões, regime e status de cada indicador (valores e cores).
  - Métricas de SL/TP (req->adj), R:R ajustado, volume por risco, além de BE/Trailing (barras decorridas e distâncias).

## 9. Suporte a vários mercados
- Tipos: Forex, Ações, Índices, Commodities, Criptomoedas e B3 (enum ENUM_MARKET_TYPE).
- O EA usa TickValue/TickSize, min/max/step de volume do símbolo e StopsLevel/FreezeLevel do broker, adaptando cálculos por ativo.
- Recomenda-se validar inputs por símbolo (StopLoss/TakeProfit/Trailing coerentes com ponto e spread).

## 10. Instalação e uso
1) Copie a pasta “EA_Oitenta_Vinte” para MQL5\Experts (e Include\ para os headers).
2) Abra no MetaEditor e compile o EA.
3) Anexe o EA ao gráfico desejado e ajuste inputs (especialmente horário, risco e módulos).
4) Opcional: ative ExecutarDiagnosticoInicial=true para um snapshot completo no início.

## 11. Boas práticas
- Inicie com ModoDebug=true e verifique se “SL/TP (req->adj) pts” e “Volume final” condizem com o esperado.
- Ajuste ThresholdConfluencia/Confiança e PorcentagemIndicadoresMinima conforme o ativo/timeframe.
- Valide AtrasoBreakEven/AtrasoTrailingStop por barras do timeframe em uso.

## 12. Troubleshooting
- SL/TP parecem “curtos”: veja “minPts” no log; aumente StopLoss/TakeProfit ou trabalhe com timeframes maiores.
- Volume por risco muito baixo/alto: revise RiskPerTradePercent e distância real do SL; confirme TickValue/TickSize do ativo.
- Sinais raros: verifique Filtro Pareto (ATR) e thresholds; use ModoFacilitado para testes.
- Mudanças de sinal fecham cedo: desative FecharNaMudancaSinal para testes comparativos.
