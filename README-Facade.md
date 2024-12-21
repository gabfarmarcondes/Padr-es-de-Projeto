# Implementação de Padrões de Projeto - Trabalho da Matéria de Engenharia de Software do curso de Ciência da Computação.

## Os Três Padrões de Projeto Escolhidos:
- Facade
- Singleton
- Chain of Responsability

### Facade
**Objetivo**: Facade é um design de padrão de projeto que ajuda a simplificar uma biblioteca, um framework ou qualquer outro set de classes complexas.

**Problema**: 
Imagine um problema que você precisa fazer seu código trabalhar com um vasto conjunto de objetos que pertencem a uma biblioteca ou um framework. Normalmente, seria necessário inicializar todos esses objectos, controlar as dependências, executar os métodos na ordem correta e assim por diante. 

Como resultado, a lógica de negócio da sua classe tornaria fortemente acoplados para a implementação detalhada da das classes terceirizadas, ficando difícil de entender e manter.

**Solução**: 
Facade é uma classe que provê uma simples interface para um complexo subsistema que contém muitas partes móveis. O facade pode fornecer funcionalidades limitadas em comparação com o trabalho do subsistema diretamente. No entanto, inclui apenas as caraterísticas que realmente interessam aos clientes. 

Ter uma fachada é útil quando precisa de integrar a sua aplicação com uma biblioteca sofisticada que tem dezenas de caraterísticas, mas só precisa de uma pequena parte da sua funcionalidade. Por exemplo, uma aplicação que carrega pequenos vídeos engraçados com gatos para as redes sociais poderia potencialmente utilizar uma biblioteca profissional de conversão de vídeo. No entanto, tudo o que realmente precisa é de uma classe com o único método “encode(filename, format)”. Depois de criar essa classe e de a ligar à biblioteca de conversão de vídeo, terá a sua primeira fachada.

<br>

**Diagrama UML**: 

![image](https://github.com/user-attachments/assets/e62c7c40-ab63-4714-a716-687b49760ab7)


**Código de Exemplo**:
```java
package refactoring_guru.facade.example.facade;

import refactoring_guru.facade.example.some_complex_media_library.*;

import java.io.File;

public class VideoConversionFacade {
    public File convertVideo(String fileName, String format) {
        System.out.println("VideoConversionFacade: conversion started.");
        VideoFile file = new VideoFile(fileName);
        Codec sourceCodec = CodecFactory.extract(file);
        Codec destinationCodec;
        if (format.equals("mp4")) {
            destinationCodec = new MPEG4CompressionCodec();
        } else {
            destinationCodec = new OggCompressionCodec();
        }
        VideoFile buffer = BitrateReader.read(file, sourceCodec);
        VideoFile intermediateResult = BitrateReader.convert(buffer, destinationCodec);
        File result = (new AudioMixer()).fix(intermediateResult);
        System.out.println("VideoConversionFacade: conversion completed.");
        return result;
    }
}
```
***Explicação***: 
Facade provê uma interface simples da conversão de vídeo. Criando um objeto da classe VideoFile e passando ele como parâmetro da calsse Codec. A classe VideoFile tem os atributos name e codecType. A interface Codec é utilizada nas classes MPEG4CompressionCodec e OggCompressionCodec. Logo, a classe CodecFactory faz a extração de áudio de cada tipo de arquivo.

O Facade foi bom pois esconde detalhes da implementação e fornece uma interface simples para um subsistema complexo, além de fácil manuntenção e escalabilidade.






REFRACTORING.GURU. Design Patterns. Disponível em: https://refactoring.guru/design-patterns/catalog. Acesso em: 21 dez. 2024.
