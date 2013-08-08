---
title: Testando componentes e controllers
layout: page
section: 15
category: [pt, docs]
---

Criar um teste unitário do seu controller VRaptor costuma ser muito fácil: dado o desacoplamento das suas classes com a API javax.servlet e os parâmetros serem populados por meio do request, seu teste será como o de uma classe qualquer, sem mistérios.

Como o VRaptor3 gerencia as dependências da sua classe, você não precisa se preocupar com a criação dos seus componentes e controllers, basta receber suas dependências no construtor que o VRaptor3 vai procurá-las e instanciar sua classe.

Na hora de testar suas classes você pode aproveitar que todas as dependências estão sendo recebidas no construtor e passar implementações falsas (mocks) dessas dependências, para testar unitariamente sua classe. No entanto, os componentes do VRaptor3 que vão ser mais presentes na sua aplicação - o Result e o Validator - possuem a interface fluente, o que dificulta a criação de implementações falsas (mocks). Por causa disso existem implementações falsas já disponíveis no VRaptor3: MockResult e MockValidator. Isso facilita em muito os seus testes que seriam mais complexos.

<h3>MockResult</h3>

O MockResult ignora os redirecionamentos que você fizer, e acumula os objetos incluídos, para você poder inspecioná-los e fazer as suas asserções.
Um exemplo de uso seria:

{% highlight java %}
MockResult result = new MockResult();
ClienteController controller = new ClienteController(..., result);
controller.list(); // vai chamar result.include("clients", algumaCoisa);
List<Client> clients = result.included("clients"); // o cast é automático
Assert.assertNotNull(clients);
// mais asserts
{% endhighlight %}

Quaisquer chamadas do tipo result.use(...) vão ser ignoradas.

<h3>MockValidator</h3>

O MockValidator vai acumular os erros gerados, e quando o validator.onErrorUse for chamado, vai lançar um ValidationError caso haja algum erro. Desse jeito, você pode inspecionar os erros adicionados, ou simplesmente ver se deu algum erro:

{% highlight java %}
@Test(expected=ValidationException.class)
public void testaQueVaiDarErroDeValidacao() {
    ClienteController controller = new ClienteController(..., new MockValidator());
    controller.adiciona(new Cliente());
}
{% endhighlight %}

ou

{% highlight java %}
@Test
public void testaQueVaiDarErroDeValidacao() {
    ClienteController controller = new ClienteController(..., new MockValidator());
    try {
        controller.adiciona(new Cliente());
        Assert.fail();
    } catch (ValidationException e) {
        List<Message> errors = e.getErrors();
        //asserts nos erros
    }
}
