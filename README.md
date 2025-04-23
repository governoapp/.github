Mapas Culturais Database Schema
Este documento descreve o esquema de banco de dados para o Mapas Culturais, implementado no Directus com PostgreSQL 17 e PostGIS. O esquema atende aos requisitos da plataforma, incluindo mapeamento colaborativo de agentes, espaços, eventos, projetos, oportunidades, avaliações, e comissões de avaliação, conforme https://cultura-ceara.gitbook.io/tutorial-para-o-usario-do-mapa-cultural/. Usuários e permissões são gerenciados nativamente pelo Directus (directus_users, directus_roles, directus_permissions).
Coleções
1. Agents
Descrição: Representa agentes culturais (individuais ou coletivos) com perfil, currículo, e geolocalização opcional.
Tabela: agents
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


user_id
M2O (directus_users)
Usuário que gerencia o agente.
Foreign Key, Nullable


name
String
Nome social do agente.
Required


type
Dropdown
Tipo do agente (individual, collective).
Required, Options: individual, collective


description
Text
Descrição detalhada do agente.



location
Geometry (Point, SRID 4326)
Coordenadas geográficas (opcional).



location_visibility
Dropdown
Visibilidade do endereço (public, private).
Default: public, Options: public, private


created_at
Timestamp
Data de criação.
Default: Now


updated_at
Timestamp
Data de última atualização.
Default: Now


metadata
JSON
Metadados flexíveis (ex.: {"website": "example.com", "social_media": {"twitter": "user"}}).
Default: {}


Configurações:

Interface: Map para location, WYSIWYG para description, JSON Editor para metadata.
Permissões: 
Admin: Full CRUD.
Editor: Create/Update próprios (user_id = $CURRENT_USER), Read todos.
User: Create/Update próprios, Read públicos.



2. Spaces
Descrição: Locais culturais (teatros, museus, bibliotecas) com geolocalização obrigatória.
Tabela: spaces
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


agent_id
M2O (agents)
Agente responsável pelo espaço.
Foreign Key, Nullable


name
String
Nome do espaço.
Required


type
Dropdown
Tipo do espaço (theater, museum, library, gallery, other).
Required, Options: theater, museum, library, gallery, other


description
Text
Descrição detalhada do espaço.



address
String
Endereço físico.



location
Geometry (Point, SRID 4326)
Coordenadas geográficas.
Required


created_at
Timestamp
Data de criação.
Default: Now


updated_at
Timestamp
Data de última atualização.
Default: Now


metadata
JSON
Metadados flexíveis (ex.: {"accessibility": "wheelchair"}).
Default: {}


Configurações:

Interface: Map para location, WYSIWYG para description, JSON Editor para metadata.
Permissões: 
Admin: Full CRUD.
Editor: Create/Update próprios (agent_id associado), Read todos.
User: Read públicos.



3. Events
Descrição: Eventos culturais com data, hora, local, e suporte a administradores adicionais.
Tabela: events
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


agent_id
M2O (agents)
Agente criador do evento.
Foreign Key, Nullable


space_id
M2O (spaces)
Espaço onde ocorre o evento (opcional).
Foreign Key, Nullable


project_id
M2O (projects)
Projeto associado (opcional).
Foreign Key, Nullable


name
String
Nome do evento.
Required


description
Text
Descrição detalhada do evento.



short_description
String
Descrição curta do evento.
Max Length: 255


start_time
Timestamp with Time Zone
Data/hora de início.
Required


end_time
Timestamp with Time Zone
Data/hora de término (opcional).
Must be > start_time if set


location
Geometry (Point, SRID 4326)
Coordenadas (opcional, se não usar space_id).



status
Dropdown
Status do evento (active, draft, trashed, archived).
Default: draft, Options: active, draft, trashed, archived


created_at
Timestamp
Data de criação.
Default: Now


updated_at
Timestamp
Data de última atualização.
Default: Now


metadata
JSON
Metadados flexíveis (ex.: {"ticket_price": "free", "age_rating": "12"}).
Default: {}


Configurações:

Interface: Map para location, Datetime para start_time/end_time, JSON Editor para metadata.
Permissões: 
Admin: Full CRUD.
Editor: Create/Update próprios (agent_id associado), Read todos.
User: Read públicos (status = 'active').



4. Projects
Descrição: Iniciativas culturais vinculadas a agentes, eventos, ou oportunidades.
Tabela: projects
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


agent_id
M2O (agents)
Agente responsável pelo projeto.
Foreign Key, Nullable


name
String
Nome do projeto.
Required


description
Text
Descrição detalhada do projeto.



start_date
Date
Data de início (opcional).



end_date
Date
Data de término (opcional).
Must be >= start_date if set


status
Dropdown
Status do projeto (active, draft, trashed, archived).
Default: draft, Options: active, draft, trashed, archived


created_at
Timestamp
Data de criação.
Default: Now


updated_at
Timestamp
Data de última atualização.
Default: Now


metadata
JSON
Metadados flexíveis (ex.: {"funding": "public"}).
Default: {}


Configurações:

Interface: Date para start_date/end_date, JSON Editor para metadata.
Permissões: 
Admin: Full CRUD.
Editor: Create/Update próprios (agent_id associado), Read todos.
User: Read públicos (status = 'active').



5. Opportunities
Descrição: Editais, chamadas, ou seleções com formulários de inscrição e avaliações.
Tabela: opportunities
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


agent_id
M2O (agents)
Agente responsável pela oportunidade.
Foreign Key, Nullable


project_id
M2O (projects)
Projeto associado (opcional).
Foreign Key, Nullable


space_id
M2O (spaces)
Espaço associado (opcional).
Foreign Key, Nullable


event_id
M2O (events)
Evento associado (opcional).
Foreign Key, Nullable


name
String
Nome da oportunidade.
Required


description
Text
Descrição detalhada da oportunidade.



start_date
Date
Data de início das inscrições.
Required


end_date
Date
Data de término das inscrições.
Required, Must be >= start_date


status
Dropdown
Status (open, closed, draft).
Default: draft, Options: open, closed, draft


form_config
JSON
Configuração do formulário (perguntas, tipos, obrigatórias).
Default: {}


created_at
Timestamp
Data de criação.
Default: Now


updated_at
Timestamp
Data de última atualização.
Default: Now


metadata
JSON
Metadados flexíveis (ex.: {"category": "music"}).
Default: {}


Configurações:

Interface: JSON Editor para form_config/metadata.
Permissões: 
Admin: Full CRUD.
Editor: Create/Update próprios (agent_id associado), Read todos.
User: Read públicos (status = 'open').



6. Registrations
Descrição: Inscrições submetidas para oportunidades, com respostas ao formulário.
Tabela: registrations
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


opportunity_id
M2O (opportunities)
Oportunidade associada.
Foreign Key, Required


agent_id
M2O (agents)
Agente que submeteu a inscrição.
Foreign Key, Nullable


number
String
Número único da inscrição (ex.: "INS-2025-001").
Unique, Required


form_data
JSON
Respostas do formulário de inscrição.
Default: {}


status
Dropdown
Status da inscrição (draft, submitted, approved, rejected).
Default: draft, Options: draft, submitted, approved, rejected


created_at
Timestamp
Data de criação.
Default: Now


updated_at
Timestamp
Data de última atualização.
Default: Now


metadata
JSON
Metadados flexíveis (ex.: {"category": "music"}).
Default: {}


Configurações:

Interface: JSON Editor para form_data/metadata.
Permissões: 
Admin: Full CRUD.
Editor: Read inscrições de oportunidades próprias, Create/Update próprias.
User: Create/Update próprias, Read submetidas.



7. Evaluations
Descrição: Avaliações (técnicas ou documentais) de inscrições, realizadas por avaliadores.
Tabela: evaluations
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


registration_id
M2O (registrations)
Inscrição avaliada.
Foreign Key, Required


agent_id
M2O (agents)
Agente avaliador.
Foreign Key, Nullable


type
Dropdown
Tipo de avaliação (technical, documentary).
Required, Options: technical, documentary


score
Float
Pontuação da avaliação (ex.: 8.5).
Min: 0


comments
Text
Comentários do avaliador.



criteria
JSON
Critérios e pontuações específicas (ex.: {"clarity": 4, "impact": 3}).
Default: {}


created_at
Timestamp
Data de criação.
Default: Now


updated_at
Timestamp
Data de última atualização.
Default: Now


Configurações:

Interface: JSON Editor para criteria.
Permissões: 
Admin: Full CRUD.
Editor: Create/Update próprias (agent_id associado), Read de oportunidades próprias.
User: Read próprias.



8. Evaluation Committees
Descrição: Comissão de avaliação de oportunidades, com avaliadores e permissões.
Tabela: evaluation_committees
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


opportunity_id
M2O (opportunities)
Oportunidade associada.
Foreign Key, Required


agent_id
M2O (agents)
Agente avaliador na comissão.
Foreign Key, Required


role
Dropdown
Papel na comissão (evaluator, coordinator).
Required, Options: evaluator, coordinator


status
Dropdown
Status do convite (pending, accepted, rejected).
Default: pending, Options: pending, accepted, rejected


created_at
Timestamp
Data de criação.
Default: Now


updated_at
Timestamp
Data de última atualização.
Default: Now


Configurações:

Permissões: 
Admin: Full CRUD.
Editor: Create/Update de oportunidades próprias, Read todos.
User: Read onde é membro (agent_id = $CURRENT_USER).



9. Files
Descrição: Arquivos (imagens, vídeos, PDFs) associados a entidades.
Tabela: files
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


entity_type
Dropdown
Tipo da entidade associada (agent, space, event, registration).
Required, Options: agent, space, event, registration


entity_id
UUID
ID da entidade associada.
Required


title
String
Título do arquivo.
Required


file
File
Arquivo (armazenado via Directus Files).
Required


type
Dropdown
Tipo do arquivo (image, video, document).
Required, Options: image, video, document


created_at
Timestamp
Data de criação.
Default: Now


Configurações:

Interface: File para file.
Permissões: 
Admin: Full CRUD.
Editor: Create/Update de entidades próprias.
User: Read públicos, Create/Update próprios.



10. Entity Relations
Descrição: Relacionamentos entre entidades (ex.: eventos em espaços, projetos com agentes).
Tabela: entity_relations
Campos:



Nome
Tipo de Dado
Descrição
Restrições



id
UUID
Chave primária.
Primary Key, Auto-generated


source_entity_type
Dropdown
Tipo da entidade origem (project, agent, space, event).
Required, Options: project, agent, space, event


source_entity_id
UUID
ID da entidade origem.
Required


target_entity_type
Dropdown
Tipo da entidade destino (agent, space, event).
Required, Options: agent, space, event


target_entity_id
UUID
ID da entidade destino.
Required


relation_type
String
Tipo de relação (member, location, occurrence, admin).
Required


created_at
Timestamp
Data de criação.
Default: Now


Configurações:

Permissões: 
Admin: Full CRUD.
Editor: Create/Update de entidades próprias.
User: Read públicos.



Notas

Usuários e Permissões: Gerenciados nativamente pelo Directus (directus_users, directus_roles, directus_permissions).
Geolocalização: Campos location usam PostGIS para consultas espaciais.
Metadados: Campos metadata e form_config usam JSON para flexibilidade.
Comissões de Avaliação: Suportadas via evaluation_committees, com notificações via directus_notifications.
Multitenancy: Adicione campo tenant_id (UUID) às coleções para suportar múltiplas instalações, com filtro:{"tenant_id": {"_eq": "$CURRENT_USER.tenant_id"}}


API: Use a API do Directus para buscas, criações, e filtros geográficos (ex.: /items/spaces?filter[location][st_dwithin]).

