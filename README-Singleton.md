# Implementação de Padrões de Projeto - Trabalho da Matéria de Engenharia de Software do curso de Ciência da Computação.

### Singleton
**Objetvio**: Singleton é um padrão de projeto criacional que permite você garantir uma única instância, enquanto provê um ponto de acesso global para essa instância.

**Problema**: 
O Singleton resolve dois problemas:
- Assegurar que a classe apenas possua uma instância: Isso para que tenha como controlar o acesso de algum recurso compartilhado, como um banco de dados.
- Prover um ponto de acesso global para essa instância: Criar variáveis globais para objetos importantes do sistema de forma insegura pode ocorrer a sobrescrita o conteúdo que ela possuia e travar toda aplicação.

**Solução**:
Toda implementação do Singleton tem esses dois padrões comuns:
- Criar um construtor padrão de forma privada, para prevenir que outros objetos utilizem o operador ```new``` em uma classe Singleton.
- Criar um método criacional estático que atua como um construtor. No fundo, este método chama o construtor privado para criar um objeto e salvar ele em um campo estático. Todas as chamadas seguintes a este método devolvem o objeto em cache.

<br>

**Diagrama UML**: 

![image](https://github.com/user-attachments/assets/ce641a4f-d462-432a-8ed1-cdab4d2923a1)

**Código de Exemplo**:
```java
package refactoring_guru.singleton.example.non_thread_safe;

public final class Singleton {
    private static Singleton instance;
    public String value;

    private Singleton(String value) {
        // The following code emulates slow initialization.
        try {
            Thread.sleep(1000);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
        this.value = value;
    }

    public static Singleton getInstance(String value) {
        if (instance == null) {
            instance = new Singleton(value);
        }
        return instance;
    }
}
```
***Explicação***:
Acima, está um Singleton Single-Threaded. Para realizar isso, basta criar uma classe, passando seus atributos e depois criar um construtor privado e implementar um método estático criacional.





REFRACTORING.GURU. Design Patterns. Disponível em: https://refactoring.guru/design-patterns/catalog. Acesso em: 21 dez. 2024.
