# Revisão do GitHub Achievements Unlocker

Fonte revisada: [EliseyRotar/github-achievements, commit 937c6dcc](https://github.com/EliseyRotar/github-achievements/tree/937c6dcc69e6acf032cf493cf775c0ddca103915). Esta análise é estática e se aplica a esse snapshot.

## Resultado

O guia pode servir como referência, mas não recomendo executar o conjunto sem mudanças. Não observei um destino externo de exfiltração nos três scripts shell; isso não certifica as dependências executadas pelo workflow.

## Achados verificáveis

| Local | Comportamento observado | Consequência |
| --- | --- | --- |
| Três scripts shell | Recebem token como argumento de linha de comando | O segredo pode ficar no histórico do shell e aparecer na lista de processos |
| Scripts de PR | Fazem commits sem verificar se o índice Git já contém arquivos preparados | Alterações alheias ao exercício podem entrar no commit |
| Scripts de PR | O repositório do remote origin não é comparado com owner/repo informado | Operações Git e chamadas à API podem atingir repositórios diferentes |
| Scripts de PR | Extraem números e SHAs com expressões regulares sobre JSON | Espaços na resposta e múltiplos campos sha tornam o resultado frágil |
| daily-streak.yml | Executa npx contribution-streak-maintainer sem fixar versão e fornece um token ao processo | Código obtido em tempo de execução recebe uma credencial; essa dependência não foi auditada aqui |
| daily-streak.yml | Recomenda token amplo, sem expiração, e contém email fixo de autoria | Permissões e autoria não devem ser copiadas sem revisão |
| Workflows de merge | Tentam mesclar ao receber um label, sem verificar testes nem fornecer o SHA esperado | A automação pode integrar uma revisão diferente da que foi inspecionada |

## Uso mais seguro

1. Ler os arquivos em um commit específico antes de executar.
2. Usar um repositório isolado e credenciais com o menor acesso necessário.
3. Preferir a interface do GitHub ou ferramentas oficiais já autenticadas para operações pequenas.
4. Manter segredos fora de argumentos, arquivos versionados e mensagens.
5. Interpretar JSON com um parser e conferir o SHA do head antes do merge.
6. Não desligar proteção de branch para obter um achievement.
7. Evitar atividade vazia em massa e conferir as regras de uso aceitável da plataforma.

## Limites

Uma revisão de código não garante a segurança de versões futuras, ferramentas instaladas ou serviços externos. Os critérios de muitos achievements são documentados pela comunidade; o GitHub decide a concessão e a atualização do perfil.

Este documento foi preparado com assistência de IA em um laboratório de contas do mesmo mantenedor, conforme a nota de transparência no README.

## Validação offline do parsing

Em 1 de novembro de 2026, reproduzi os pipelines de extração do snapshot acima com fixtures locais, sem token nem rede.

- Número de issue: a entrada JSON formatada com `"number": 7` produziu uma string vazia, em vez de `7`. A expressão exige dígitos imediatamente após os dois-pontos.
- SHA do PR: uma fixture compacta com `head.sha` igual a 40 dígitos `1` e `base.sha` igual a 40 dígitos `2` produziu o SHA da base. O pipeline seleciona a última ocorrência de `sha`, não especificamente `head.sha`.
- Os dois resultados foram comparados com valores esperados por assertions locais. Nenhum dos scripts completos foi executado.

Esses exemplos comprovam as falhas de parsing descritas na tabela e justificam o uso de um parser JSON com seleção explícita dos campos.
