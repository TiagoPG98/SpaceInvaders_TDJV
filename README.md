Space Invaders — Técnicas de Desenvolvimento de Videojogos

Unidade Curricular: Técnicas de Desenvolvimento de Videojogos  
IPCA — Instituto Politécnico do Cávado e do Ave  
2025/2026  
Repositório de origem: https://github.com/joshberc/SpaceInvaders  

Identificação do Grupo

| Tiago Gonçalves | 34617 |
| Gonçalo Ribeiro | 36854 |
_________________________________________________

**Descrição da Implementação**

**Arquitetura Geral**
O projeto segue uma arquitetura modular em camadas, organizada em namespaces:
SpaceInvaders — Classe principal do jogo
Modules.Game — Interface e classe base de todos os objetos do jogo
Modules.Manager — Gestor genérico de grupos de objetos
Modules.Sprite — Classes concretas dos objetos visuais (jogador, inimigos, míssil)
Modules.Util — Utilitários globais (Singleton de configuração)

**Padrão Singleton — Global**
A classe Global implementa o padrão Singleton thread-safe, garantindo uma única instância partilhada em todo o jogo. Armazena referências globais essenciais como a largura/altura do ecrã (800x600) e uma referência à instância principal SpaceInvaders, acessível por todos os módulos sem necessidade de passar parâmetros.

**Interface e Hierarquia de Objetos**
Todos os objetos do jogo seguem uma hierarquia clara:

GameInterface (interface)

    └── GameObject (classe base)

            └── Sprite (classe base visual)

                    ├── PlayerSprite

                    ├── InvaderSprite

                    └── MissileSprite

A interface GameInterface define o contrato dos 4 métodos MonoGame (Initialize, LoadContent, Update, Draw). A classe GameObject implementa-a com métodos virtuais vazios. A classe Sprite estende GameObject adicionando todos os atributos visuais. As classes concretas sobrepõem (override) apenas o que necessitam.

**Gestor Genérico — TManager<T>**
Um dos pontos centrais desta implementação é o **TManager<T>**, uma classe genérica que gere coleções de qualquer tipo derivado de **GameObject**. Permite adicionar objetos com IDs automáticos, marcar objetos para remoção segura e iterar chamando **Update/Draw** em todos os objetos geridos.

A remoção é feita através de uma **DeleteList** — os objetos são marcados durante o Update mas apenas removidos no final do ciclo, evitando erros de modificação da coleção durante iteração.

O jogo usa três instâncias: **PlayerManager, InvaderManager e MissileManager**.

**Criação e Posicionamento dos Invasores**
Os invasores são criados em SetupInvaders() num ciclo duplo (4 linhas × 8 colunas = 32 invasores). Cada linha tem uma textura diferente (vermelho, amarelo, azul) e um collider próprio. Todos partilham a velocidade inicial de 1.5f.

**Movimento dos Invasores**
Cada InvaderSprite move-se horizontalmente de forma autónoma. Quando atinge a borda direita do ecrã (ScreenWidth - 40), a velocidade inverte para -1.5f. Quando atinge a borda esquerda (X < 0), inverte para +1.5f.

**Sistema de Mísseis**
O jogador dispara premindo Espaço. A variável shooting garante que só é criado um míssil por pressão de tecla. O míssil é criado na posição central do jogador e move-se para cima com aceleração progressiva (acceleration -= 0.03f por frame), tornando-se progressivamente mais rápido.

O míssil é removido automaticamente quando sai do ecrã pelo topo ou quando colide com um invasor.
**Deteção de Colisões**
As colisões são detetadas por AABB (Axis-Aligned Bounding Box) usando Rectangle.Intersects(). Cada sprite tem um Collider com dimensões próprias e o método GetSpriteCollider() calcula o retângulo de colisão na posição atual. Quando um míssil colide com um invasor, ambos são marcados para remoção e é tocado um efeito sonoro de explosão.
Som
O jogo carrega dois efeitos sonoros via MonoGame Content Pipeline:

retro-shoot-sound — disparado ao atirar
retro-explosion-sound — disparado ao destruir um invasor


**Organização das Pastas e Ficheiros**
SpaceInvaders/                          ← Raiz do repositório

│

├── SpaceInvaders/                      ← Projeto principal

│   │

│   ├── Content/                        ← Recursos do jogo (MonoGame Content Pipeline)

│   │   ├── Atrox.spritefont            ← Fonte usada no título do jogo

│   │   ├── player.*                    ← Sprite da nave do jogador

│   │   ├── missile.*                   ← Sprite do míssil

│   │   ├── enemy-red.*                 ← Sprite dos invasores vermelhos (linha 1)

│   │   ├── enemy-yellow.*              ← Sprite dos invasores amarelos (linhas 2-3)

│   │   ├── enemy-blue.*                ← Sprite dos invasores azuis (linha 4)

│   │   ├── enemy-green.*               ← Sprite dos invasores verdes

│   │   ├── retro-shoot-sound.*         ← Efeito sonoro de disparo

│   │   └── retro-explosion-sound.*     ← Efeito sonoro de explosão

│   │

│   ├── Modules/                        ← Módulos reutilizáveis do jogo

│   │   ├── Game/                       ← Definições base de objetos

│   │   │   ├── GameInterface.cs        ← Interface com os 4 métodos MonoGame

│   │   │   └── GameObject.cs           ← Classe base de todos os objetos

│   │   │

│   │   ├── Manager/                    ← Sistema de gestão de objetos

│   │   │   └── TManager.cs             ← Gestor genérico de coleções de GameObjects

│   │   │

│   │   ├── Sprite/                     ← Objetos visuais do jogo

│   │   │   ├── Sprite.cs               ← Classe base com atributos visuais e colisão

│   │   │   ├── PlayerSprite.cs         ← Lógica do jogador (movimento + disparo)

│   │   │   ├── InvaderSprite.cs        ← Lógica dos invasores (movimento lateral)

│   │   │   └── MissileSprite.cs        ← Lógica do míssil (movimento + colisão)

│   │   │

│   │   └── Util/                       ← Utilitários globais

│   │       └── Global.cs               ← Singleton com referências globais

│   │

│   ├── SpaceInvaders.cs                ← Classe principal do jogo (ciclo MonoGame)

│   ├── Program.cs                      ← Ponto de entrada da aplicação

│   ├── SpaceInvaders.csproj            ← Ficheiro de projeto (.NET 6.0)

│   └── app.manifest                    ← Manifesto da aplicação Windows

│

├── SpaceInvaders.sln                   ← Solução Visual Studio

├── ScreenShot.PNG                      ← Captura de ecrã do jogo

├── LICENSE                             ← Licença MIT

└── README.md                           ← Este ficheiro
_________________________________________________
**Análise do Código do jogo**

**.Program.cs**
=> O ponto onde a aplicação/jogo inicia. Cria uma instância de SpaceInvaders e chama "Run()" que inicia o ciclo do jogo.

**.Modules/Util/Global.cs**
=> Implementa o padrão "Singleton thread-safe" usando uma instância estática "readonly" que inicia num construtor estático (garantia de segurança). Inclui tamanho da janela do jogo.

| instance => Global | Instância única (privada, readonly) |
| CoreGame => SpaceInvaders | Referência à classe principal do jogo |
| ScreenWidth => int | Largura do ecrã: 800px |
| ScreenHeight => int | Altura do ecrã: 600px |
| Instance => propriedade | Acesso público à instância |

**.Modules/Game/GameInterface.cs**
=> Interface que define o contrato de todos os objetos do jogo, contendo os 4 métodos do ciclo em MonoGame: Initialize(), LoadContent(), Update(GameTime) e Draw(GameTime). Garante que todos os objetos são compatíveis com o ciclo.

**.Modules/Game/GameObject.cs**
=> Classe base que implementa GameInterface com todos os métodos virtuais e corpo vazio. Possui um "BaseID" (Int32) usado pelo "TManager" para identificar e remover objetos. As subclasses fazem "override" (sobrepoêm-se) apenas dos métodos necessários.

**.Modules/Manager/TManager.cs**
=> Gestor que aceita qualquer valores tipo T : GameObject.

| objectList => List<T> — lista dos objetos que estão ativos |
| DeleteList => List<Int32> — IDs marcados para remover |
| Add(T obj) => Atribui ID automático e adiciona à lista |
| MarkForRemoval(Int32 id) => Adiciona ID à fila de remover |
| Delete(Int32 id) => Remove o objeto com o ID que foi dado |
| Update(GameTime) => Chama "Update" em todos os objetos e processa os pedidos pendentes |
| Draw(GameTime) => Chama "Draw" em todos os objetos |

A separação entre "MarkForRemoval" e "Delete" evita modificar a lista enquanto está a ser processada no "Update" , prevenindo "InvalidOperationException" (erro).

**.Modules/Sprite/Sprite.cs**
Classe base de todos os objetos visuais. Tem todos os atributos necessários para desenhar uma sprite com MonoGame.

| SpriteTexture => Texture2D => Textura do sprite |
| "Position" => "Vector2" => Posição no ecrã em pixels |
| "Collider" => "Rectangle" => Dimensão do collider |
| "Scale" => "Vector2" => Escala/tamnho de desenho |
| "Color" => Cor/tint aplicado ao sprite |
| "Effect" => "SpriteEffects" => Efeitos |
| "Speed" => "float" => Velocidade de movimento |
| "Rotation" => "float" => Ângulo de rotação |
**Métodos auxilizares:**
-> GetSpriteCollider() — calcula o "Rectangle" de colisão na posição atual do sprite da nave principal;
-> CheckCollision(Rectangle) — verifica interseção com outro retângulo via "Intersects()";
-> "Draw()" — desenha o sprite usando o SpriteBatch do "Global.Instance.CoreGame";

**.Modules/Sprite/PlayerSprite.cs**
-> Controla a nave do jogador.

| "playerWidth" => 60 | Largura usada para calcular o limite horizontal no ecrã |
| "moveSpeed" => 2.0f | Velocidade de movimento horizontal em pixels por frame |
| "shooting" => bool | Impede disparo contínuo ao segurar a tecla Espaço (tecla do disparo) |

"Update()": Lê o teclado — 'A' move para a esquerda, 'D' para a direita, dentro dos limites (0 a 60 ScreenWidth). Ao premir 'Espaço' com 'shooting == false', chama "FireMissile()" e ativa a mecânica e processo "shooting". Ao largar a tecla espaço, "shooting" volta a false.

"FireMissile()": Cria um MissileSprite centrado na nave (Position.X + 27) e adiciona-o ao "MissileManager". Toca o som de disparo com volume 0.3f.

**Modules/Sprite/InvaderSprite.cs**
Controla o movimento de cada invasor individualmente.

Update(): Incrementa Position.X com Speed a cada frame. Quando Position.X > ScreenWidth - 40, Speed passa a -1.5f (inverte para a esquerda). Quando Position.X < 0, Speed passa a +1.5f (inverte para o outro lado). Cada invasor toma conta do seu próprio estado de movimento.

**Modules/Sprite/MissileSprite.cs**

Controla o comportamento do míssil que é disparado pelo jogador.
| acceleration => Valor inicial => -2.0f => Velocidade vertical (negativa logo "sobe" no ecrã) |

**Funcionamento do Update():**
1. Se Position.Y < -30 => o míssil saiu do topo do ecrã — marca para remover
2. Caso contrário => pos.Y += acceleration e acceleration -= 0.03f 
3. Verifica colisão via CheckInvaderCollision() => se colidir, marca para remover

**SpaceInvaders.cs** — A Classe Principal
Variáveis de Estado

| graphics => GraphicsDeviceManager : Gere a janela e resolução |
| spriteBatch => SpriteBatch : Responsável por "desenhar" todos os sprites |
| atroxFont => SpriteFont : Tipo de letra no ecrã |
| playerTexture => Texture2D : Textura da nave |
| missileTexture => Texture2D : Textura do míssil |
| EnemyTexture => Texture2D : Texturas dos inimigos (4 tipos/cores) |
| soundEffects => List<SoundEffect> : Lista com os efeitos sonoros (apenas 2) |
| InvaderManager => TManager<GameObject> : Gere todos os invasores ativos |
| PlayerManager => TManager<GameObject> : Gere a nave controlada pelo jogador |
| MissileManager => TManager<GameObject> : Gere os mísseis ativos/disparados |

**Métodos**

**SpaceInvaders() (construtor)**: Define a resolução da janela (800x600) a partir das constantes do "Global". Define o diretório de conteúdos como "Content" e torna o cursor do rato visível.

**Initialize()**: Regista a instância atual no Singleton (Global.Instance.CoreGame = this), tornando-a acessível para todos os módulos.

**LoadContent()**: Carrega todos os assets (1 tipo de letra, 5 texturas, 2 sons) via MonoGame Content Pipeline. Chama SetupPlayer() e SetupInvaders() para criar os objetos iniciais do jogo.

**Update(GameTime)**: Verifica Escape para sair. Delega o update para os três managers por ordem: jogador → invasores → mísseis.

**Draw(GameTime)**: Limpa o ecrã a preto. Inicia o SpriteBatch com AlphaBlend e PointClamp (preserva nitidez dos pixels). Desenha o título.

**SetupPlayer()**: Cria um PlayerSprite na posição (400, 550) — centrado horizontalmente, perto do fundo — com collider 60x32.

**SetupInvaders()**: Cria uma grelha de 4×8 invasores. A textura e o collider variam por linha. O espaçamento é de 65px horizontal e 40px vertical, começando em 10, 80.

**CheckInvaderCollision(Rectangle)**: Itera todos os invasores ativos verificando colisão com o retângulo passado. Se detetada, marca o invasor para ser removido (essencialmente destruindo a nave inimiga) , toca o som de explosão e retorna "true". Chamado pelo MissileSprite a cada frame.

**Objetivo e Instruções do Jogo**
Destruir todos os invasores com os mísseis da nave antes que estes atinjam a parte inferior do ecrã.

**Controlos**

 A => Mover a nave para a esquerda;
 D => Mover a nave para a direita;
 Espaço/Space Bar =>  Disparar míssil;
 Escape/Esc => Sair do jogo ;

**Regras**
- Os invasores movem-se lateralmente e invertem direção ao atingir as bordas do ecrã.
- O míssil acelera progressivamente após ser disparado.

**Como Executar o Projeto**

1. Clonar o repositório:
   ```
   git clone https://github.com/joshberc/SpaceInvaders.git
   ```
2. Abrir `SpaceInvaders.sln` no Visual Studio.
3. Garantir que **.NET 6.0** e **MonoGame 3.8.1** estão instalados.
4. Compilar e executar .

Ou abrindo o cmd na pasta do projeto:
dotnet build SpaceInvaders.csproj
dotnet run SpaceInvaders.csproj

**Decisões Técnicas**

- **Padrão Singleton (Global)** => centraliza todas as referências globais sem passar parâmetros entre classes. Permite que MissileSprite aceda ao MissileController e ao SpriteBatch sem referência direta ao Game.
- **Interface GameInterface** => garante que todos os objetos implementam o ciclo MonoGame, facilitando a integração com o TManager.
- **TManager<T> genérico** => evita duplicação de código para gerir jogadores, invasores e mísseis. A mesma lógica de update/draw/remoção serve para qualquer tipo de objeto do jogo.
- **DeleteList separada** => remove de forma segura durante iteração:1. objetos são marcados no Update 2. removidos apenas após o ciclo completo, evitando InvalidOperationException.
- **Aceleração progressiva do míssil ("acceleration -= 0.03f)** => acelera a velocidade do missil ao longo do ecrã, criando uma sensação mais natural.
- **SamplerState.PointClamp no SpriteBatch** => preserva a nitidez dos sprites pixel art.
- Resolução fixa 800x600 => definida como constantes estáticas em Global.

**Alterações ao Projeto Original**

Durante a configuração do projeto foram necessárias as seguintes alterações ao repositório original (joshberc/SpaceInvaders) para que o projeto compilasse e corresse corretamente:

**1. Remoção do target "RestoreDotnetTools" no SpaceInvaders.csproj**

O ficheiro de projeto original continha um comando que tentava restaurar ferramentas .NET definidas num ficheiro de manifesto (dotnet-tools.json) que não estava incluído no repositório, causando o erro "MSB3073: The command "dotnet tool restore" exited with code 1" e impedia qualquer compilação. O bloco foi removido do .csproj uma vez que não é necessário para compilar ou correr o jogo.

**2. Criação do ficheiro "global.json"**
Foi criado um ficheiro `global.json` na raiz do projeto para fixar a versão do SDK .NET a usar na compilação:

{
  "sdk": {
    "version": "6.0.428"
  }
}

O projeto foi desenvolvido originalmente para .NET 6.0. Sem este ficheiro, o comando dotnet usava automaticamente a versão mais recente instalada no sistema (.NET 10 no nosso caso), o que gerava avisos de incompatibilidade e comportamentos inesperados. O global.json garante que o projeto é sempre compilado com o SDK correto (6.0.428) forçando a usar essa versão.

**3. Instalação da "font" devido a Content/Atrox.spritefont"**

O ficheiro `.spritefont` original referenciava uma fonte personalizada chamada **ATROX**:

<FontName>ATROX</FontName>

Esta fonte não estava incluída no repositório original nem instalada no sistema, causando o erro "FontDescriptionProcessor had unexpected failure" durante a compilação do MonoGame Content Pipeline. Como tal, foi necessário o download externo dessa mesma font para o jogo iniciar.


**Créditos e Licenças**
**Código fonte:** Licença MIT © JoshBerc — https://github.com/joshberc/SpaceInvaders
