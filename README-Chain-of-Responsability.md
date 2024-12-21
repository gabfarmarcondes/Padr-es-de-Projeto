# Implementação de Padrões de Projeto - Trabalho da Matéria de Engenharia de Software do curso de Ciência da Computação.

## Chain of Responsibility

**Objetivo**: É um padrão de projeto comportamental que permite que você passe requisições ao longo de uma cadeia de manipuladores. Ao receber a requisição, cada manipulador decide processar a requisição ou passar para o próximo manipulador da cadeia

**Problema**: 
Imagina que você está trabalhando em um sistema ordenação online. Você quer restringir o acesso para o sistema tenha somente uma autenticação de usuários que podem criar a ordenação. Além disso, os utilizadores com permissões administrativas devem ter acesso total a todas as encomendas.

Depois de um planejamento, você percebe que estas verificações devem ser efetuadas sequencialmente. A aplicação pode tentar autenticar um usuário no sistema sempre que receber um pedido que contenha as credenciais do usuário. No entanto, se essas credenciais não estiverem corretas e a autenticação falhar, não há razão para prosseguir com quaisquer outras verificações.

Durante os próximos meses, você implementa alguns sequencias de verificações.
- Um dos seus colegas sugere que isso é inseguro, passar esses dados de forma ordenada no sistema, então você cria outra autenticação.
- Depois, alguém percebe que o sistema é vulnerável para a quebra de senhas por força bruta. Então é adicionado outra autenticação.
- Outra pessoa sugeriu que se poderia acelerar o sistema devolvendo resultados em cache em pedidos repetidos contendo os mesmos dados. Por isso, adicionou outra verificação.

  O código de verificação, que até o momento está uma bagunça, tornou-se cada vez mais sobrecarregado à medida que adicionava cada nova funcionalidade.

  O sitema ficou imcompreendível e difícil de manter.

**Solução**: Como muitos padrões de projeto comportamentais, o Chain of Responsability se baseia na transformação de comportamentos particulares em objetos independentes chamados manipuladores. No nosso caso, cada verificação deve ser extraída para a sua própria classe com um único método que executa a verificação. O pedido, juntamente com seus dados, é passado para este método como um argumento.

O padrão sugere que você ligue esses manipuladores em cada cadeia. Cada manipulador ligado tem um campo para guardar uma referência para o próximo manipulador da cadeia. Além disso, o processo de requisição dos manipuladores passa a requisição para frente através da cadeia. A requisiçao viaja ao longo das cadeias até o manipulador ter a chance de o processar.


**Diagrama UML**:
<br>
![image](https://github.com/user-attachments/assets/d044cd38-2c4f-467b-a1f8-03eb04433ada)


**Código de Exemplo**:
```java
public abstract class Middleware {
    private Middleware next;

    public static Middleware link(Middleware first, Middleware... chain) {
        Middleware head = first;
        for (Middleware nextInChain: chain) {
            head.next = nextInChain;
            head = nextInChain;
        }
        return first;
    }

    public abstract boolean check(String email, String password);

    protected boolean checkNext(String email, String password) {
        if (next == null) {
            return true;
        }
        return next.check(email, password);
    }
}
```
***Explicação***: Esta classe base para todas os manipuladores da cadeia. Tendo um atributo ```next```que aponta para a próxima parte da cadeia. O método link é responsável por ligar as cadeias. O método check é abstrato e deve ser implementado pelas classes concretas. Por fim, o método checkNext chama o próximo da cadeia.

```java
public class ThrottlingMiddleware extends Middleware {
    private int requestPerMinute;
    private int request;
    private long currentTime;

    public ThrottlingMiddleware(int requestPerMinute) {
        this.requestPerMinute = requestPerMinute;
        this.currentTime = System.currentTimeMillis();
    }

    public boolean check(String email, String password) {
        if (System.currentTimeMillis() > currentTime + 60_000) {
            request = 0;
            currentTime = System.currentTimeMillis();
        }

        request++;

        if (request > requestPerMinute) {
            System.out.println("Request limit exceeded!");
            Thread.currentThread().stop();
        }
        return checkNext(email, password);
    }
}
```
***Explicação***: Este manipulador verifica se o número de requisições feitas em um minuto excedeu o limite definido. Se o limite for ultrapassado, ele interrompe o processo.

```java
public class UserExistsMiddleware extends Middleware {
    private Server server;

    public UserExistsMiddleware(Server server) {
        this.server = server;
    }

    public boolean check(String email, String password) {
        if (!server.hasEmail(email)) {
            System.out.println("This email is not registered!");
            return false;
        }
        if (!server.isValidPassword(email, password)) {
            System.out.println("Wrong password!");
            return false;
        }
        return checkNext(email, password);
    }
}
```
***Explicação***:Este manipulador verifica se o e-mail está registrado no servidor e se a senha é válida para o e-mail fornecido.

```java
public class RoleCheckMiddleware extends Middleware {
    public boolean check(String email, String password) {
        if (email.equals("admin@example.com")) {
            System.out.println("Hello, admin!");
            return true;
        }
        System.out.println("Hello, user!");
        return checkNext(email, password);
    }
}
```
***Explicação***: Este manipulador verifica se o usuário é um administrador, imprimindo uma mensagem diferente com base no tipo de usuário.

```java
public class Server {
    private Map<String, String> users = new HashMap<>();
    private Middleware middleware;

    public void setMiddleware(Middleware middleware) {
        this.middleware = middleware;
    }

    public boolean logIn(String email, String password) {
        if (middleware.check(email, password)) {
            System.out.println("Authorization have been successful!");
            return true;
        }
        return false;
    }

    public void register(String email, String password) {
        users.put(email, password);
    }

    public boolean hasEmail(String email) {
        return users.containsKey(email);
    }

    public boolean isValidPassword(String email, String password) {
        return users.get(email).equals(password);
    }
}
```
***Explicação***: A classe Server possui um map de usuários e o atributo middleware que mantém a cadeia de manipuladores. O método logIn recebe as credenciais do usuário e passa para a cadeia de manipuladores através do método check.

```java
public class Demo {
    private static BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
    private static Server server;

    private static void init() {
        server = new Server();
        server.register("admin@example.com", "admin_pass");
        server.register("user@example.com", "user_pass");

        Middleware middleware = Middleware.link(
            new ThrottlingMiddleware(2),
            new UserExistsMiddleware(server),
            new RoleCheckMiddleware()
        );

        server.setMiddleware(middleware);
    }

    public static void main(String[] args) throws IOException {
        init();

        boolean success;
        do {
            System.out.print("Enter email: ");
            String email = reader.readLine();
            System.out.print("Input password: ");
            String password = reader.readLine();
            success = server.logIn(email, password);
        } while (!success);
    }
}
```
***Explicação***: A classe Demo inicializa o servidor e registra dois usuários. Ela também cria a cadeia de manipuladores, configurando o servidor para usar essa cadeia ao realizar o login.

<br>
REFRACTORING.GURU. Design Patterns. Disponível em: https://refactoring.guru/design-patterns/catalog. Acesso em: 21 dez. 2024.
