title: Java EE 8 MVC
date: 2016/03/21 12:00
authorId: VKO
tags: [java, mvc, java ee, spring]
---

Začátkem roku 2014 proběhl průzkum, ve kterém se komunita měla vyjádřit k tomu, co by se mělo stát součástí Java EE 8. Více než 60% respondentů odpovědělo, že Java EE by mělo podrorovat vedle JSF i jinou formu MVC pro webové aplikace [[1]](#Survey).
Následně započaly práce na JSR-371 nazvaném MVC: Model-View-Controller API [[2]](#JSR371). A právě na nové MVC API se v tomto článku podíváme.

<!-- more -->
 
## K čemu to je

Můžete namítnout, že v JEE už jeden MVC framework fro webové aplikace máme: JSF. Autoři vysvětlují, že se jedná o trochu jiné MVC.
JSF je zaměřeno na vytváření komponent, které jsou využívány IoC frameworkem. Nové MVC API ponechává vývojářům větší kontrolu, neboť aplikační kód funguje na úrovni HTTP požadavků a odpovědí - prostě tak, jak to známe ze Spring MVC.
Pro podrobnější vysvětlení viz [[3]](#JSF).

## Začínáme psát kód

Vydání Java EE 8 je plánováno až na první pololetí roku 2017, ale my si můžeme nové API vyzkoušet již nyní. Práce na referenční implementaci jsou totiž v plném proudu a i když do finální verze specifikace jistě dojde ke změnám, základní principy zůstanou zachovány.
Zmíněná implementace nese název Ozark a je ke stažení zde: [[4]](#Ozark). Pokud používáte Gradle, stačí přidat do build.gradle

```groovy
    compile group:'org.glassfish.ozark', name: 'ozark', version: '1.0.0-m02'
```

a můžeme začít vyvíjet. Pro spuštění potřebujeme samozřejmě vhodný JEE server. Autoři Ozarku testují své dílo s GlassFish 4.1 Nightly Sept 15, 2015, takže my použijeme totéž [[6]](#GlassFish).

## Controller

Autoři specifikace MVC API nezačínali na zelené louce a nepokoušeli se znovu vynalézat kolo. Specifikace využívá CDI a v podstatě pouze rozšiřuje existující specifikaci JAX-RS. Pokud se podíváme na typický controller pro operace CRUD, je to jasně vidět:

```java
@Controller @Path("projects")
public class ProjectController {

    @Inject
    Models models;

    @Inject
    ProjectRepository projectRepository;

    @GET
    public String list() {
        models.put("projects", projectRepository.findAll());
        return "projects.jsp";
    }

    @Path("/new") @GET
    public String showCreate() {
        return "editProject.jsp";
    }

    @Path("/new") @POST
    public String create(@BeanParam Project project) {
        projectRepository.save(project);
        return "redirect:projects";
    }

    @Path("/{id}") @GET
    public String get(@PathParam("id") Integer id) {
        models.put("project", projectRepository.get(id));
        return "editProject.jsp";
    }

    @Path("/{id}") @POST
    public String update(@BeanParam Project project) {
        projectRepository.save(project);
        return "redirect:projects";
    }

    @Path("/{id}/remove") @GET
    public String delete(@PathParam("id") Integer id) {
        projectRepository.delete(id);
        return "redirect:projects";
    }

}
```

Většina anotací v controlleru je převzatá z JAX-RS:

{% img  center-block /attachments/2016-03-21-java-ee-mvc/jeemvcjaxrs.png 400 %}

Model je do controlleru injectován pomocí CDI. Takže jedinými novinkami jsou
- anotace Controller, která oznamuje, že třída je MVC controllerem
- rozhraní Models, což je pouze mapa, kterou controller naplní hodnotami. Tyto hodnoty jsou potom k dispozici ve view
- návratová hodnota typu String, která identifikuje view, které bude použito pro renderování modelu.

Zdrojové kody celé demo aplikace včetně view jsou kdispozici zde: [[6]](#Demo)

## Model a controller

Controller má dvě možnosti, jak předat model do view:
1. Pomocí mapy "Models", jak je to popsáno výše.
2. Naplněním pojmenovaného (@Named) beanu, který je potom do view injectován pomocí CDI. Implementace MVC nemusí tuto možnost podporovat, je pouze doporučená.

Na view není nic zvláštního. Implementace musí podporovat JSP a Facelty, další technologie (například Thymeleaf) budou volitelné.
View pro metodu `list()` z našeho controlleru může vypadat například takto:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<html>
  <head>
    <title>Projects</title>
  </head>
  <body>
    <h1>Projects</h1>
    <ul>
      <c:forEach items="${projects}" var="project">
        <li><a href="projects/${project.id}">${project.id} - ${project.name}</a></li>
      </c:forEach>
    </ul>
    <a href="projects/new">Create</a>
  </body>
</html>

```

## Srovnání se Spring MVC

Jestliže znáte Spring MVC, nové JEE MVC API vás ničím nepřekvapí. Následující obrázek hovoří za vše:

{% img  center-block /attachments/2016-03-21-java-ee-mvc/comparison.png 800 %}

## Závěr

Nové MVC je něco, co ve standardním JEE chybělo. Sympatická je snaha maximálně využít existující specifikace (především JAX-RS) a stavět na nich.
Uvidíme, jak se nové API rozšíří, až se objeví první servery, které budou implementaci JSR-371 obsahovat.

## Odkazy
- <a name="Survey">[1]</a> Results from the Java EE 8 Community Survey [https://java.net/downloads/javaee-spec/JavaEE8_Community_Survey_Results.pdf](https://java.net/downloads/javaee-spec/JavaEE8_Community_Survey_Results.pdf)
- <a name="JSR371">[2]</a> JSR 371: Model-View-Controller (MVC 1.0) Specification [https://www.jcp.org/en/jsr/detail?id=371](https://www.jcp.org/en/jsr/detail?id=371)
- <a name="JSF">[3]</a> Why Another MVC? [http://www.oracle.com/technetwork/articles/java/mvc-2280472.html](http://www.oracle.com/technetwork/articles/java/mvc-2280472.html)
- <a name="Ozark">[4]</a> Ozark - Reference Implementation for MVC 1.0. [https://ozark.java.net/download.html](https://ozark.java.net/download.html)
- <a name="GlassFish">[5]</a> GlassFish 4.1 Nightly Sept 15, 2015 [http://download.oracle.com/glassfish/4.1/nightly/glassfish-4.1-b17-09_15_2015.zip](http://download.oracle.com/glassfish/4.1/nightly/glassfish-4.1-b17-09_15_2015.zip)
- <a name="Demo">[6]</a> Java MVC Demo Application [https://github.com/vitkoma/java-mvc-demo](https://github.com/vitkoma/java-mvc-demo)