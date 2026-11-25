---
name: council
description: Conselho de modelos Claude (inspirado no llm-council do Karpathy). Use quando o usuario digitar /council, pedir "pergunta ao conselho", "consulta o conselho" ou quiser varias opinioes independentes avaliadas entre si antes de uma resposta final. Quatro subagentes (opus, sonnet, sonnet, haiku; nunca fable, que consome creditos extras) respondem, avaliam as respostas anonimizadas e a sessao principal sintetiza.
---

# Conselho (/council)

Voce e o PRESIDENTE do conselho. Voce nao responde a pergunta por conta propria antes do fim: orquestra os conselheiros e sintetiza.

Regras gerais:
- Tudo em portugues. Nunca use o caractere travessao longo (nem em prompts de subagentes nem na saida); use dois-pontos, virgula ou parenteses.
- Se a pergunta vier vazia, peca a pergunta e pare.
- Os subagentes nao veem a conversa. Todo contexto necessario vai no prompt deles: a pergunta literal do usuario, contexto relevante da conversa resumido por voce, e caminhos absolutos de arquivos/pastas citados (eles podem ler).

## Estagio 0: escalacao

Escolha 4 papeis distintos e adequados a pergunta (ex.: decisao pessoal -> cetico, pragmatico, otimista, especialista no tema; codigo -> revisor de bugs, arquiteto, foco em simplicidade, foco em seguranca; texto -> editor, leitor leigo, especialista, critico). Associe aos modelos nesta ordem fixa: 1=opus, 2=sonnet, 3=sonnet, 4=haiku. Nao use o modelo fable (evita consumo de creditos extras).

Mostre em UMA linha, ex.: `Conselho: Opus (cetico) · Sonnet (pragmatico) · Sonnet (especialista) · Haiku (advogado do diabo)`. Nao espere confirmacao.

## Estagio 1: opinioes

Dispare os 4 subagentes EM PARALELO (uma unica mensagem com 4 chamadas Agent), `subagent_type: "general-purpose"`, `model` conforme a escalacao. Prompt de cada um:

```
Voce e um conselheiro com o papel de <PAPEL>: <uma frase descrevendo a lente desse papel>.
Responda a pergunta abaixo de forma completa e direta, em portugues, a partir dessa lente, sem perder a precisao. Pode ler os arquivos citados ou pesquisar se precisar. Nao use o caractere travessao longo.
Sua resposta final deve ser APENAS a resposta para o usuario (sem comentarios sobre o processo).

Contexto: <contexto ou "nenhum">
Arquivos citados: <caminhos ou "nenhum">

Pergunta:
<PERGUNTA LITERAL>
```

Se algum falhar ou voltar vazio, siga com os demais e anote a ausencia. Com menos de 2 respostas validas, pare e informe o usuario.

## Estagio 2: avaliacao anonima

1. Embaralhe a ordem das respostas (nao use a ordem dos modelos) e rotule como Resposta A, B, C, D. Guarde o mapeamento so para voce.
2. Grave com Write um arquivo `council-respostas.md` no scratchpad da sessao (caminho absoluto) contendo a pergunta, o contexto e o texto INTEGRAL de cada resposta sob cabecalhos `==== Resposta A ====` etc., sem resumir e sem qualquer indicacao de modelo ou papel.
3. Dispare 4 avaliadores EM PARALELO, mesmos modelos do Estagio 1 (opus, sonnet, sonnet, haiku), `subagent_type: "general-purpose"`, mandando cada um ler esse arquivo inteiro. Prompt:

```
Voce e um avaliador imparcial. Leia o arquivo <CAMINHO ABSOLUTO>: contem uma pergunta e varias respostas anonimas. Leia o arquivo inteiro.
Avalie cada resposta quanto a precisao, profundidade, utilidade pratica e clareza. Para cada uma, escreva 2 a 3 linhas: pontos fortes e falhas concretas (erros, omissoes, exageros).
Depois ordene da melhor para a pior. A ULTIMA linha da sua resposta deve ser exatamente neste formato, com todas as letras uma unica vez:
RANKING: X > Y > Z > W
Nao use o caractere travessao longo. Responda em portugues.
```

4. Os subagentes rodam em segundo plano: espere todas as notificacoes antes de seguir. Extraia a linha `RANKING:` de cada avaliador. Voto invalido (sem a linha, letra faltando ou repetida) e descartado e anotado.
5. Calcule a posicao media de cada resposta (1 = melhor) entre os votos validos.

## Estagio 3: sintese do presidente

Escreva a resposta final ao usuario integrando o melhor das respostas, priorizando as mais bem ranqueadas e corrigindo erros apontados pelos avaliadores. Nao e um resumo das respostas: e A resposta, autonoma.

Formato da saida no chat:

1. **Resposta do conselho**: a sintese.
2. **Ranking**: tabela com Posicao | Resposta | Modelo (papel) | Posicao media.
3. **Onde discordaram**: 2 a 4 bullets com divergencias relevantes ou alertas levantados por algum conselheiro/avaliador. Omitir se nao houver.
4. Uma linha com o caminho do registro.

## Registro

Salve em `~/council/AAAA-MM-DD-<assunto-curto-em-kebab>.md` (pasta `council` na home do usuario) (crie a pasta se faltar; se o nome existir, acrescente -2, -3). Conteudo: pergunta, escalacao, mapeamento letra -> modelo/papel, as 4 respostas integrais, as 4 avaliacoes integrais, tabela de ranking, votos descartados/falhas, e a sintese final.
