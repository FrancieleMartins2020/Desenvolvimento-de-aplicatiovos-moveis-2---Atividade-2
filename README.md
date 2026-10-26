# 🛒 Gerenciador de Compras (MVVM + SQLite)

Sistema robusto corporativo para gerenciamento de listas de compras segmentadas por locais e departamentos específicos de lojas. O projeto foi arquitetado sob os mais rígidos critérios de Engenharia de Software modernos, utilizando o padrão **MVVM pura** com injeção de dependência e persistência em banco de dados relacional local.

## 📐 Arquitetura do Sistema (MVVM)

A estrutura adota uma separação rígida de responsabilidades distribuída em camadas explícitas:

1.  **View (Apresentação):** Telas puras desprovidas de lógica de dados. Apenas renderizam o layout escutando o gerenciador de estado.
2.  **ViewModel (Estado Reativo):** Intermedeia a View e o Repositório, expondo propriedades reativas estendendo `ChangeNotifier`.
3.  **Repository (Domínio/Regras):** Consolida regras de consulta, evita redundâncias e abstrai a fonte dos dados.
4.  **Service (Persistência):** Conecta de forma bruta com a API relacional do SQLite.

## 🚀 Funcionalidades Críticas e Implementação

### 1. Suporte Nativa para Windows Desktop (FFI Bridge)
Como o driver clássico do SQLite (`sqflite`) é exclusivo para ambientes móveis (Android/iOS), foi implementada uma ponte em nível de sistema operacional no `DatabaseService` através do `sqflite_common_ffi`. A inicialização do banco injeta as funções `sqfliteFfiInit()` e muda o `databaseFactory` global antes do build da interface.

### 2. Integridade Referencial em Cascata
Para blindar o banco de dados contra registros órfãos ou lixo eletrônico na memória do Windows, o aplicativo ativa a verificação nativa de chaves estrangeiras com `PRAGMA foreign_keys = ON;`. A tabela de itens utiliza a diretriz `ON DELETE CASCADE`, fazendo com que a remoção de uma lista apague automaticamente no mesmo milissegundo todos os produtos associados a ela.

### 3. Filtro Geral e Algoritmo de Reordenação
*   **Filtro Inclusivo:** A listagem possui um seletor onde a flag nula (`null`) representa dinamicamente a opção "Todos os Departamentos", listando os itens gerais sem divisão.
*   **Reordenação na CPU:** Para poupar processamento e Queries no disco rígido, a mudança de estado de um item (marcado como comprado) dispara um `UPDATE` veloz e aplica um algoritmo nativo `.sort()` em memória, jogando os itens comprados automaticamente para o fim da lista e aplicando o estilo riscado (`TextDecoration.lineThrough`).

## 🛠️ Tecnologias e Pacotes Utilizados

*   **Framework:** Flutter (Null-Safety)
*   **Gerenciamento de Estado:** `provider: ^6.1.2` (MVVM Binding)
*   **Persistência Relacional:** `sqflite: ^2.3.5` e `path: ^1.9.0`
*   **Desktop Driver:** `sqflite_common_ffi: ^2.3.4` (FFI Bridge)

## 📁 Estrutura de Pastas Exigida

```text
lib/
├── data/
│   ├── database/
│   │   └── database_service.dart      # Inicialização FFI / Criação de Tabelas
│   ├── models/
│   │   ├── setor_model.dart           # Entidade de Setores
│   │   ├── lista_compra_model.dart    # Entidade de Listas
│   │   └── item_model.dart            # Entidade de Itens/Produtos
│   └── services/
│       └── storage_service.dart       # Contrato de Interface da Persistência
├── repositories/
│   └── shopping_repository.dart       # Queries e Regras de Negócio do SQLite
├── viewmodels/
│   ├── main_view_model.dart           # Estado das Listas Gerais e Setores
│   └── shopping_view_model.dart       # Estado dos Itens / Filtros e Ordenação
├── views/
│   ├── main_list_view.dart            # Tela Inicial de Gerenciamento
│   ├── setores_management_view.dart   # Tela de Cadastro de Setores
│   └── shopping_details_view.dart     # Tela do Carrinho / Execução da Compra
└── main.dart                          # Inicializador Global (MultiProvider)
```
