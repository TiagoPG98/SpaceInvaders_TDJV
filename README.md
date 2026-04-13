Space Invaders — Técnicas de Desenvolvimento de Videojogos

Unidade Curricular: Técnicas de Desenvolvimento de Videojogos  
IPCA — Instituto Politécnico do Cávado e do Ave  
2025/2026  
Repositório de origem: https://github.com/joshberc/SpaceInvaders  

Identificação do Grupo

| Tiago Gonçalves | 34617 |
| Gonçalo Ribeiro | 36854 |
_________________________________________________


_________________________________________________
Análise do Código do jogo

.Program.cs
-> O ponto onde a aplicação/jogo inicia. Cria uma instância de SpaceInvaders e chama "Run()" que inicia o ciclo do jogo.

.Modules/Util/Global.cs
->Implementa o padrão "Singleton thread-safe" usando uma instância estática "readonly" que inicia num construtor estático (garantia de segurança). Inclui tamanho da janela do jogo.

| instance => Global | Instância única (privada, readonly) |
| CoreGame => SpaceInvaders | Referência à classe principal do jogo |
| ScreenWidth => int | Largura do ecrã: 800px |
| ScreenHeight => int | Altura do ecrã: 600px |
| Instance => propriedade | Acesso público à instância |

.Modules/Game/GameInterface.cs
-> Interface que define o contrato de todos os objetos do jogo, contendo os 4 métodos do ciclo em MonoGame: Initialize(), LoadContent(), Update(GameTime) e Draw(GameTime). Garante que todos os objetos são compatíveis com o ciclo.

.Modules/Game/GameObject.cs
-> Classe base que implementa GameInterface com todos os métodos virtuais e corpo vazio. Possui um "BaseID" (Int32) usado pelo "TManager" para identificar e remover objetos. As subclasses fazem "override" (sobrepoêm-se) apenas dos métodos necessários.

.Modules/Manager/TManager.cs
-> Gestor que aceita qualquer valores tipo T : GameObject.

| objectList => List<T> — lista dos objetos que estão ativos |
| DeleteList => List<Int32> — IDs marcados para remover |
| Add(T obj) => Atribui ID automático e adiciona à lista |
| MarkForRemoval(Int32 id) => Adiciona ID à fila de remover |
| Delete(Int32 id) => Remove o objeto com o ID que foi dado |
| Update(GameTime) => Chama "Update" em todos os objetos e processa os pedidos pendentes |
| Draw(GameTime) => Chama "Draw" em todos os objetos |

A separação entre "MarkForRemoval" e "Delete" evita modificar a lista enquanto está a ser processada no "Update" , prevenindo "InvalidOperationException" (erro).

.Modules/Sprite/Sprite.cs
Classe base de todos os objetos visuais. Tem todos os atributos necessários para desenhar uma sprite com MonoGame.

| SpriteTexture => Texture2D => Textura do sprite |
| "Position" => "Vector2" => Posição no ecrã em pixels |
| "Collider" => "Rectangle" => Dimensão do collider |
| "Scale" => "Vector2" => Escala/tamnho de desenho |
| "Color" => Cor/tint aplicado ao sprite |
| "Effect" => "SpriteEffects" => Efeitos |
| "Speed" => "float" => Velocidade de movimento |
| "Rotation" => "float" => Ângulo de rotação |
Métodos auxilizares:
-> GetSpriteCollider() — calcula o "Rectangle" de colisão na posição atual do sprite da nave principal
-> CheckCollision(Rectangle) — verifica interseção com outro retângulo via "Intersects()"
-> "Draw()" — desenha o sprite usando o SpriteBatch do "Global.Instance.CoreGame"

.Modules/Sprite/PlayerSprite.cs
-> Controla a nave do jogador.

| "playerWidth" => 60 | Largura usada para calcular o limite horizontal no ecrã |
| "moveSpeed" => 2.0f | Velocidade de movimento horizontal em pixels por frame |
| "shooting" => bool | Impede disparo contínuo ao segurar a tecla Espaço (tecla do disparo) |

"Update()": Lê o teclado — 'A' move para a esquerda, 'D' para a direita, dentro dos limites (0 a 60 ScreenWidth). Ao premir 'Espaço' com 'shooting == false', chama "FireMissile()" e ativa a mecânica e processo "shooting". Ao largar a tecla espaço, "shooting" volta a false.

"FireMissile()": Cria um MissileSprite centrado na nave (Position.X + 27) e adiciona-o ao "MissileManager". Toca o som de disparo com volume 0.3f.
