# Caso 01 — Conflito entre regra de validação legada e nova integração de pagamento

## Contexto

Um sistema de retaguarda de varejo precisava integrar um novo meio de pagamento eletrônico. Esse novo meio dependia de identificar de forma única cada "espécie de documento" de pagamento — mas o sistema já tinha uma regra de validação (vou chamar de RV-03) que garantia unicidade de espécie de documento em outro nível, criada anos antes para outro propósito.

A nova integração exigia unicidade composta (espécie + adquirente), enquanto a regra existente exigia unicidade simples (só espécie). Rodar as duas regras juntas, sem ajuste, quebrava um dos dois fluxos.

## Diagnóstico

O primeiro instinto da equipe de desenvolvimento foi tratar isso como bug pontual e sugerir uma exceção no código para o novo meio de pagamento. Ao investigar o histórico da regra RV-03, ficou claro que ela não era acidental — protegia contra um erro real (`NonUniqueResultException`) em métodos de consulta que dependiam da unicidade simples em várias partes do sistema, não só no ponto de conflito.

Criar uma exceção isolada teria resolvido o sintoma e reintroduzido o risco de erro em produção em pontos não testados.

## Decisão

Em vez de uma exceção pontual, a decisão foi redesenhar a regra para unicidade composta (espécie + adquirente) em todos os pontos que a consultavam, mapeando previamente cada método afetado (`carrega()`, `carregaValeCompra()`, entre outros) antes de alterar o comportamento.

Trade-off assumido: mais esforço de mapeamento e testes upfront, em troca de eliminar o risco de quebra silenciosa em fluxos não diretamente relacionados à nova integração.

## Resultado

A mudança evitou reincidência do erro em métodos que não faziam parte do escopo original da integração, mas que dependiam da mesma regra. O processo também gerou um mapeamento de dependências da regra RV-03 que passou a ser referência para mudanças futuras nesse ponto do sistema — reduzindo o tempo de análise de impacto em alterações seguintes.

---

*Nomes de sistemas, empresas e integrações específicas foram generalizados. A lógica do problema e da decisão é real.*
