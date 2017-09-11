# Metaprogramación

## Open class

Se puede agregar comportamiento a una clase pre-existente volviendo a declarar la clase con nuevos métodos:

```ruby
class Brocoli
end

class Brocoli
  def agregar_chocolate
    puts "curioso pero apetitoso"
  end

  def agregar_salsa_blanca
    puts "ñom ñom"
  end

  def agregar_ajo
    puts "siempre se puede poner más ajo"
  end
end
```

## Singleton class

Internamente, cada instancia tiene asociada una Singleton Class.
En esta clase es donde quedan registrados los métodos propios de dicha instancia.

```ruby
  unBrocoli = Brocoli.new
  def unBrocoli.comer
    puts "solamente este brocoli se puede comer"
  end

  # este mensaje funciona porque la Singleton Class de unBrocoli
  # tiene la definición de 'comer'
  unBrocoli.comer

  # este mensaje fallará porque sólo la instancia referenciada por
  # 'unBrocoli' tiene deinido 'comer'
  # Brocoli.new.comer 
```

## Send

Podemos enviar mensajes dinámicamente utilizando `send` y un symbol como argumento.

```ruby
  %w(chocolate salsa_blanca ajo)
      .map { |ingrediente| "agregar_#{ingrediente}" }
      .each { |method| Brocoli.new.send(method.to_sym) }
```

## Method Missing

Toda clase puede sobreescribir `method_missing` y responder a mensajes que normalmente no entendería.
En lugar de ejecutar el comportamiento default (lanzar una excepción) es posible recibir el nombre del mensaje recibido y
argumentos y actuar.

```ruby
  class ObjetoEnfuruñado
    def method_missing(sym, *args, &block)
      puts "No voy a #{sym.to_s.split('_').join(' ')}"
    end
  end

  # En lugar de romperse con una excepción por no entender los mensajes
  # ejecuta el código en method_missing, mostrando un texto por pantalla.
  ObjetoEnfuruñado.new.pintar_la_cerca
  ObjetoEnfuruñado.new.sacar_la_basura

```

## Define Method

Utilizando `define_method` podemos definirle un nuevo método a una clase.

```ruby
    unaClase.define_method("hola") do |aQuien|
      puts "hola #{aQuien}"
    end
```
Dentro de la definición de la clase, self es la misma clase, con lo cual
el receptor de define_method en el siguiente ejemplo sigue siendo esa clase.

```ruby
  class MetaBrocoli

    { agregar_chocolate: "curioso pero apetitoso",
      salsa_blanca: "ñom ñom",
      ajo:"siempre se puede poner más ajo"
    }.each { |ingrediente, frase|
      define_method("agregar_#{ingrediente}") do
        |terminacion, a, b|
        puts frase + terminacion
      end
    }
```

Debe tenerse en cuenta que `define_method` es un método privado, con lo cual
para enviarlo desde fuera de la declaración de la clase debe combinarse con `send`, ya que `send`
puede enviar mensajes incluso si el método es privado.

## Ejercicio

Se quiere poder instanciar un objeto Diccionario
que permita agregar dinámicamente atributos de manera similar a
como se hace en JavaScript:

```ruby
objeto.nombre = "pepe"
puts objeto.nombre #-> "pepe"
```
Si intento acceder a un field que no fue seteado previamente, se debe lanzar una excepción.
Debo poder setear una cantidad ilimitada de fields.

#### Solución: utilizando un diccionario interno

```ruby
class Diccionario

  def initialize
    @dic = {}
  end
  def method_missing(sym, *args, &block)
    if sym.to_s.end_with? '='
      @dic[sym.to_s] = args[0]
    else
      @dic[sym.to_s + '=']
    end
  end
end
```

#### Solución: utilizando `instance_variable_set` e `instance_variable_get`

```ruby
class Diccionario
  def method_missing(sym, *args, &block)
    if sym.to_s.end_with? '='
      instance_variable_set "@#{sym.to_s.chop}", args[0]
    else
      instance_variable_get "@#{sym.to_s}"
    end
  end
end
```

#### Solución: utilizando `define_method`

```ruby
class Diccionario
  def method_missing(sym, *args, &block)
    unless sym.to_s.end_with? '='
      super
      return
    end

    self.class.send(:define_method, sym.to_s.chop.to_sym){
      args[0]
    }
  end
end
```
