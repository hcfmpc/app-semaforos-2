1. Declaração de Missão:
Disponibilizar uma aplicação mobile que, a partir de mapa GPS e dados de semáforos, oriente o motorista sobre a velocidade ideal (média, mínima e máxima) para atravessar o maior número possível de sinais verdes no trajeto atual, reduzindo consumo de combustível e emissão de CO₂.  
O aplicativo deve obter os ciclos semafóricos por integração com base de dados externa; na ausência desses dados, deve permitir cadastro colaborativo dos horários de abertura/fechamento por cruzamento.

2. Personas:
- Motorista urbano diário (casa–trabalho), foco em previsibilidade e economia.
- Motorista profissional (táxi/app/frota), foco em eficiência operacional.
- Motorista eventual em área urbana, foco em orientação simples e segura.

3. Histórias de Usuário:
- Como motorista urbano, quero visualizar minha posição em um mapa GPS em tempo real para entender meu contexto de navegação.
- Como motorista urbano, quero receber recomendação de velocidade média, mínima e máxima para o próximo semáforo, para aumentar a chance de passar no verde.
- Como motorista urbano, quero receber orientação para a sequência dos próximos semáforos, para manter fluidez no trajeto.
- Como motorista colaborador, quero cadastrar ciclos semafóricos de cruzamentos sem dados, para melhorar a base da região.
- Como motorista, quero saber quando a recomendação não é confiável, para dirigir com segurança.

4. Regras de Negócio Invariantes:
- O sistema nunca deve recomendar ação que implique avanço de sinal vermelho.
- Toda recomendação de velocidade deve respeitar o limite legal da via.
- Recomendações só podem ser exibidas com dados mínimos válidos: posição GPS, direção da via, distância ao semáforo e fase/ciclo do sinal.
- Na ausência de dados confiáveis, a recomendação deve ser suspensa e o usuário informado claramente.
- Cadastros manuais de ciclos semafóricos devem registrar usuário, timestamp e nível de confiança (ex.: pendente/validado).
- Alterações colaborativas devem ser auditáveis e reversíveis.
- O app deve priorizar segurança viária sobre otimização de tempo/combustível.

5. Cenários de Falha:
- Timeout/erro de API de mapas ou semáforos: aplicar retentativa com limite; persistindo falha, usar último dado em cache com aviso de desatualização.
- Perda de sinal GPS ou baixa precisão: pausar recomendações até recuperação mínima de precisão.
- Dados semafóricos inconsistentes ou conflitantes: marcar cruzamento como “não confiável” e não gerar recomendação automática.
- Falta de conexão: operar em modo degradado (mapa/ciclos em cache) e bloquear novos cadastros online até reconexão.
- Erros de cadastro pelo usuário: validar faixas de horário e formato antes de salvar; exibir mensagem de correção.