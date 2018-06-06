---
layout: post
title:  "PHP, Traits e Singleton"
date:   2018-06-06 07:00:00 -0300
---


A minha maior motivação em participar do [fórum iMasters](imasters-perfil), sempre foi ajudar, ensinar e aprender. O
último, como sempre,
é o maior ganho que se pode ter, aprender enquanto ajuda alguém com dificuldade. Dentro desse contexto, algumas
situações acabam nos dando uma lição ainda mais surpreendente.

A partir de um questionamento ([Qual a utilidade da traits][imasters-utilidade-traits]), me vi intrigado em demonstrar
exemplos práticos de quão poderosa as traits são. Além disso, como dizia tio Ben, "_grandes poderes requerem grandes
responsabilidades_".

## Traits

O [manual][php-traits] é bem claro quanto a motivação das traits:
> Traits are a mechanism for code reuse in single inheritance languages such as PHP. A Trait is intended to reduce some
limitations of single inheritance by enabling a developer to reuse sets of methods freely in several independent classes
living in different class hierarchies. The semantics of the combination of Traits and classes is defined in a way which
reduces complexity, and avoids the typical problems associated with multiple inheritance and Mixins.

Parafraseando e simplificando, `traits` são um mecanismo de reutilização de código para linguagens de herança
simples/única. Dessa forma, é uma adição a herança tradicional habilitando a composição horizontal de
comportamento.

Uma `trait` pode agrupar um conjunto de funcionalidades e estas serem introduzidas em outras classes ou objetos. Assim,
esta classes irão encorporar o comportamento da `trait`, da mesma forma que se o código da `trait`. Isso se dá devido
ao funcionamento do interpretador para com a `trait`. Ele realmente *copia e cola* o código da `trait` dentro da classe
em que ela foi definida.

## Singleton

[`Singleton`][singleton-definition] é um padrão de projeto que define que uma classe deve possuir apenas uma instância
e um ponto de acesso global.

Normalmente, o padrão, é muito mal utilizado. Mas, sempre há a sua aplicabilidade. Um dos mais famosos uso é o
[`Registry`][registry-definition].

`Registry` é um repositórios de objetos. Normal﻿mente ele é aplicado como um padrão Singleton (mas não necessariamente
deve ser um).

Dessa forma, vamos a implementação de um Registry:

{% highlight php %}
class Registry
{

    /**
     * @var $this
     **/
    protected static $instance;

    /**
     * @var mixed[]
     **/
    private $registry = array();

    /**
     * Retrieve a value by his key
     * @param string $key.
     * @return mixed.
     * @throws \InvalidArgumentException If the key isn't registred
     **/
    public function get($key)
    {
        if (!isset($this->registry[$key])) {
            throw new InvalidArgumentException(
                sprintf("There\'s no value for key %s." , $key)
            );
        }

        return $this->registry[$key];
    }

    /**
     * Checks if a key is setted in registry
     * @param string $key
     * @return boolean
     */
    public function has($key)
    {
        return isset($this->registry[$key]);
    }

    /**
     * Define a pair key=value in registry
     * @param string $key.
     * @param mixed $value
     **/
    public function set($key , $value)
    {
        $this->registry[$key] = $value;
    }

    public static function getInstance()
    {
        return isset(static::$instance)
               ? static::$instance
               : static::$instance = new static();
    }

    /**
     * Final private constructor to prevent creating a new instance of the
     * *Singleton* via the `new` operator from outside of this class
     * and for not extend override the method
     */
    private function __construct()
    {
    }

    /**
     * Private unserialize method to prevent unserializing of the *Singleton*
     * instance.
     *
     * @return void
     */
    private function __wakeup()
    {
    }

    /**
     * Private clone method to prevent cloning of the instance of the
     * *Singleton* instance.
     *
     * @return void
     */
    private function __clone()
    {
    }
}
{% endhighlight %}

O `Registry`, nesse exemplo, é um `Singleton`. Os seguintes pontos são específicos do `Singleton`:
- `__construct`: `privado` - não permitir instância direta;
- `__wakeup`: `private` - não permitir unserialize;
- `__clone`: `private` - não permitir clone da instância;
- `getInstance`: `public` - retornar a instância única;
- `$instance`: `private` e `static` - manter a instância única.


Por outro lado, os demais métodos são específicos do `Registry`:
- `get`: retornar um registro;
- `set`: inserir um registro;
- `has`: verificar se um registro existe.

Suponha que eu precise de outra classe que implemente o `Singleton` (um manipulador para o DOM, por exemplo). Logo, eu
preciso implementar todos os métodos básicos novamente. Vamos ao molde da nossa próxima classe Singleton:

{% highlight php %}
class SingletonClass
{

    /**
     * @var $th﻿is
     **/
    protected static $instance;

    public static function getInstance()
    {
        return isset(static::$instance)
               ? static::$instance
               : static::$instance = new static();
    }

    /**
     * Final private constructor to prevent creating a new instance of the
     * *Singleton* via the `new` operator from outside of this class
     * and for not extend override the method
     */
    private function __construct()
    {
    }

    /**
     * Private unserialize method to prevent unserializing of the *Singleton*
     * instance.
     *
     * @return void
     */
    private function __wakeup()
    {
    }

    /**
     * Private clone method to prevent cloning of the instance of the
     * *Singleton* instance.
     *
     * @return void
     */
    private function __clone()
    {
    }
}
{% endhighlight %}


O código acima exemplifica o básico para uma classe `Singleton`. Qualquer classe que necessitar ser um
`Singleton`, deverá implementar o código acima.

Como é de praxe, sabe-se que copiar e colar não é uma boa alternativa e, nesse caso, devemos encontrar uma maneira para
que o código seja reutilizado.

Outro ponto importante, é que uma classe `Singleton` não atua muito bem com herança (na realidade, ela inibe o uso da
herança por trazer para si o controle da instância). Dessa forma, não é interessante implementar o `Singleton` como uma
classe abstrata. Apesar de ser totalmente possível, apenas não é aconselhado.

Eis que surgem as `traits`. Nesse caso, é possível separar em uma `trait`, toda a implementação do `Singleton`.

{% highlight php %}
namespace Harbinger\StandardLibrary\Traits;

/**
 * Trait for ﻿singleton
 * @package Harbinger
 * @subpackage StandardLibrary\Trait
 * @author Gabriel Heming <gabriel.heming@hotmail.com>
 **/
trait Singleton
{

    protected static $instance;

    final public static function getInstance()
    {
        return isset(static::$instance) ? static::$instance : static::$instance = new static();
    }

    /**
     * Final private constructor to prevent creating a new instance of the
     * *Singleton* via the `new` operator from outside of this class
     * and for not extend override the method
     */
    final private function __construct()
    {
        $this->init();
    }

    /**
     * this method must be used as constructor
     */
    protected function init()
    {
    }

    /**
     * Private unserialize method to prevent unserializing of the *Singleton*
     * instance.
     *
     * @return void
     */
    final private function __wakeup()
    {
    }

    /**
     * Private clone method to prevent cloning of the instance of the
     * *Singleton* instance.
     *
     * @return void
     */
    final private function __clone()
    {
    }
}
{% endhighlight %}

As únicas diferenças da implementação via `trait` são: a adição da _keyword_ `final`, para a classe que implementar
o `Singleton` não sobrescreva o método implementado, e a implementação do método `init`, que pode ser usado como
construtor de uma classe.

Dessa forma, a classe `Registry` pode ser reescrita da seguinte maneira:

{% highlight php %}
namespace Harbinger\StandardLibrary;

/**
 * Implementation of a data registry.
 * @package Harbinger
 * @subpackage StandardLibrary
 * @author Gabriel Heming <gabriel.﻿heming@hotmail.com.br>
 **/
class Registry
{
    use \Harbinger\StandardLibrary\Traits\Singleton;

    /**
     * @var mixed[]
     **/
    private $registry = array();

    /**
     * Retrieve a value by his key
     * @param string $key.
     * @return mixed.
     * @throws \InvalidArgumentException If the key isn't registred
     **/
    public function get($key)
    {
        if (!isset($this->registry[$key])) {
            throw new InvalidArgumentException(
                sprintf("There\'s no value for key %s." , $key)
            );
        }

        return $this->registry[$key];
    }

    /**
     * Checks if a key is setted in registry
     * @param string $key
     * @return boolean
     */
    public function has($key)
    {
        return isset($this->registry[$key]);
    }

    /**
     * Define a pair key=value in registry
     * @param string $key.
     * @param mixed $value
     **/
    public function set($key , $value)
    {
        $this->registry[$key] = $value;
    }
}
{% endhighlight %}

Assim, foi utilizada uma `trait` (que realiza o _copy and paste_) para evitar o "copia e cola" de códigos através de
vários arquivos.

De qualquer forma, esse é apenas um dos muitos exemplos da aplicabilidade de uma trait, algumas das quais irei trazer
em breve.

[perfil-imasters]: https://forum.imasters.com.br/profile/159859-gabriel-heming
[imasters-utilidade-traits]: https://forum.imasters.com.br/topic/551196-qual-a-utilidade-da-traits/
[php-traits]: http://php.net/manual/en/language.oop5.traits.php
[singleton-definition]: https://en.wikipedia.org/wiki/Singleton_pattern
[registry-definition]: https://www.martinfowler.com/eaaCatalog/registry.html