# claude-council

Um "conselho" de modelos Claude dentro do Claude Code, inspirado no [llm-council](https://github.com/karpathy/llm-council) do Andrej Karpathy.

Em vez de perguntar a um modelo só, o `/council` faz a pergunta passar por quatro conselheiros, cada um com um papel diferente, que depois avaliam as respostas uns dos outros sem saber quem escreveu. No fim, a sessão principal faz o papel de presidente e entrega uma resposta única.

Não usa API externa nem chave: roda só com subagentes do próprio Claude Code, dentro da sua assinatura.

## Como funciona

1. **Escalação:** o presidente escolhe 4 papéis adequados à pergunta (ex.: cético, pragmático, especialista, advogado do diabo) e associa aos modelos Opus, Sonnet, Sonnet e Haiku.
2. **Opiniões:** os 4 conselheiros respondem em paralelo, sem ver os outros.
3. **Avaliação anônima:** as respostas são embaralhadas e renomeadas (A, B, C, D). Quatro avaliadores dão notas e terminam com `RANKING: X > Y > Z > W`. O presidente tira a posição média.
4. **Síntese:** o presidente escreve a resposta final, mostra o ranking (revelando qual modelo e papel deu cada resposta) e os pontos de divergência.

A sessão completa (respostas e avaliações integrais) fica salva em `~/council/AAAA-MM-DD-assunto.md`.

## Instalação

**Como plugin** (dentro do Claude Code):

```
/plugin marketplace add ivanlorandi/claude-council
/plugin install council@claude-council
```

Instalado como plugin, o comando fica `/council:council`.

**Manual:** copie a pasta `plugins/council/skills/council` para `~/.claude/skills/council`. Assim o comando fica só `/council`.

## Uso

```
/council Vale mais a pena guardar dados de drone em HD externo ou no Google Drive?
```

Pode citar arquivos ou pastas na pergunta: os conselheiros recebem os caminhos e podem ler.

## Custo

Cada pergunta dispara 8 subagentes (4 conselheiros e 4 avaliadores). Não há cobrança extra de API, mas consome bem mais do limite de uso do plano do que uma pergunta normal. Reserve para decisões que valem a pena. O modelo Fable fica de fora de propósito, para não gerar créditos extras.

## Personalizar

Tudo está em texto em `SKILL.md`: dá para trocar os modelos, o número de conselheiros, os papéis sugeridos ou o idioma editando o arquivo.

## Licença

MIT
