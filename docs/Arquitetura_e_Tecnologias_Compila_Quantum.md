**Arquitetura e Tecnologias Previstas para Implementação** 

**1\. Arquitetura Proposta** 

A plataforma Compila Quantum será desenvolvida como uma aplicação web cliente-servidor organizada em camadas. Considerando o porte acadêmico do projeto e os requisitos levantados, propõe-se que o núcleo da aplicação siga uma estrutura de monólito modular, evitando a fragmentação desnecessária em diversos microsserviços. Nesse núcleo ficarão concentradas as regras relacionadas a usuários, trilha de aprendizagem, módulos, capítulos, conteúdos, exercícios, desafios, submissões e administração. A exceção será o mecanismo responsável pela execução de código e simulação quântica, que deverá permanecer isolado por possuir requisitos técnicos e de segurança diferentes das demais funcionalidades. 

Essa separação está de acordo com o modelo de domínio apresentado no diagrama de classes, que relaciona entidades como User, Client, Admin, Module, Chapter, Study, Exercises, Challenge e Submission. Também atende aos casos de uso e User Stories que preveem uma sequência organizada de estudos, exercícios associados aos conteúdos, desafios progressivos e repetíveis, acompanhamento do progresso, administração dos materiais e registro das submissões realizadas. 

A camada de apresentação será responsável pelas interfaces utilizadas pelo estudante e pelo administrador. O front-end não terá acesso direto ao banco de dados nem à infraestrutura que executará os códigos submetidos. Toda operação será intermediada pela API, que aplicará as regras de negócio, realizará autenticação e autorização, controlará a persistência e decidirá quando uma solicitação deverá ser encaminhada ao ambiente de simulação. 

Para funcionalidades convencionais, o fluxo previsto será: 

**● React/TypeScript → ASP.NET Core Web API → PostgreSQL.** Já as atividades e desafios práticos terão um fluxo diferente: 

**● Usuário → React/TypeScript → ASP.NET Core Web API → componente isolado de execução → Python/Qiskit → resultado → API → interface de visualização.** 

1  
**2\. Tecnologias e Justificativas** 

O front-end será desenvolvido com React e TypeScript. A escolha está relacionada ao caráter interativo das telas previstas nos wireframes. A plataforma possui trilha de aprendizagem, exercícios integrados ao conteúdo, editor de código e componentes que deverão ser atualizados após cada execução. Uma arquitetura baseada em componentes permitirá separar essas partes da interface e reutilizá-las entre as diferentes telas. 

O gerenciamento reativo de estado também será importante no Sandbox, pois uma modificação no circuito ou no código poderá produzir novos resultados, exigindo a atualização do console, das probabilidades e das representações gráficas sem reconstruir toda a página. O TypeScript acrescentará verificação estática de tipos à camada cliente, contribuindo para que estruturas recebidas da API sejam manipuladas de maneira consistente. 

O back-end principal será implementado em C\# com ASP.NET Core Web API. Essa camada concentra autenticação e autorização, regras de negócio, operações administrativas, gestão dos conteúdos, exercícios e desafios, acompanhamento do progresso e registro das submissões. A tipagem estática de C\# favorece a modelagem explícita das entidades e dos Data Transfer Object (DTOs) utilizados na comunicação com o front-end, enquanto o ASP.NET Core oferece uma estrutura adequada para APIs e operações assíncronas, característica relevante principalmente quando a aplicação precisar aguardar ou acompanhar uma solicitação de simulação. 

Embora .NET disponha de estruturas como System.Numerics.Complex, isso não significa que o mecanismo de Computação Quântica deva ser desenvolvido em C\#. A responsabilidade do back-end será principalmente controlar o domínio da plataforma e orquestrar as execuções, enquanto os cálculos e simulações quânticas serão delegados a um ambiente especializado. 

A persistência utilizará PostgreSQL, pois os dados principais possuem relacionamentos bem definidos. Um banco relacional permitirá representar os vínculos com chaves, restrições e integridade referencial, evitando inconsistências no histórico acadêmico do usuário. 

2  
O Entity Framework Core será utilizado como ORM entre C\# e PostgreSQL, realizando o mapeamento das entidades do domínio, seus relacionamentos e consultas, além de permitir que alterações no esquema sejam controladas por migrations. Recursos como JSON/JSONB poderão ser utilizados de maneira complementar para resultados ou metadados de simulação cuja estrutura seja variável, mas não substituirão o modelo relacional das entidades centrais. 

Para a parte quântica, propõe-se um componente implementado em Python com Qiskit e Qiskit Aer. Essa decisão decorre principalmente do ecossistema científico disponível em Python e do suporte específico do Qiskit à criação, manipulação e simulação dos circuitos. Esse ambiente poderá processar informações quânticas (qubits, portas, medições, vetores de estado e etc). Os resultados serão convertidos para uma estrutura apropriada à comunicação com a API, permitindo que o front-end apresente as informações do estado quântico. Quando houver amplitudes complexas, por exemplo, elas serão serializadas em uma representação compatível com a comunicação entre os componentes, como pares de valores real e imaginário. 

**3\. Integração e Execução do Sandbox** 

O código produzido no Sandbox será considerado entrada não confiável e, portanto, não deverá ser executado dentro do mesmo processo da ASP.NET Core Web API. Caso isso ocorresse, um código com consumo excessivo de recursos, execução prolongada ou operações indevidas poderia comprometer a aplicação principal. 

Por esse motivo, recomenda-se utilizar Docker ou mecanismo equivalente para criar ambientes de execução isolados e descartáveis. Nesse contexto, sua função não será apenas facilitar a implantação da aplicação: cada execução poderá ocorrer em um ambiente controlado, com versão definida de Python e das bibliotecas do Qiskit, limites de CPU e memória, tempo máximo de execução, usuário sem privilégios e restrição de acesso à rede quando ela não for necessária. 

A API encaminhará a solicitação para um componente executor interno, que iniciará o ambiente apropriado, capturará de maneira controlada stdout, stderr, estado da execução, tempo utilizado e resultados da simulação. O executor devolverá essas informações de forma estruturada à API, que poderá registrar os 

3  
dados relevantes na entidade de submissão e posteriormente encaminhá-los ao front-end. 

Assim, a arquitetura manterá separadas duas responsabilidades distintas: a ASP.NET Core Web API será responsável pelo negócio e pelo controle da plataforma, enquanto Python/Qiskit será responsável pela execução e simulação quântica em ambiente isolado. Essa organização atende às funcionalidades apresentadas nos requisitos e protótipos sem introduzir a complexidade de uma arquitetura completa de microsserviços, mantendo uma solução compatível com a dimensão e os objetivos acadêmicos do projeto. 

4