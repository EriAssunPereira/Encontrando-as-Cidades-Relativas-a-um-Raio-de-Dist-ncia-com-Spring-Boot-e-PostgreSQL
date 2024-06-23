# Encontrando-as-Cidades-Relativas-a-um-Raio-de-Distância-com-Spring-Boot-e-PostgreSQL

Para evoluir uma API de cidades em Java utilizando Spring Boot e PostgreSQL para encontrar cidades dentro de um raio de distância, vamos dividir o projeto em algumas partes principais: configuração do ambiente, modelagem de dados, desenvolvimento da API REST, e implementação do cálculo de distância.

### Passos do Projeto

#### Parte 1: Configuração do Ambiente

1. **Configuração do Projeto Spring Boot**
   - Crie um novo projeto Spring Boot utilizando o Spring Initializr (https://start.spring.io/).
   - Adicione as dependências necessárias como `Spring Web`, `Spring Data JPA`, `Spring Boot DevTools`, etc.
   - Configure o projeto para usar o PostgreSQL como banco de dados.

2. **Configuração do Banco de Dados**
   - Configure as propriedades de conexão com o PostgreSQL no arquivo `application.properties`:
     ```properties
     spring.datasource.url=jdbc:postgresql://localhost:5432/nome_do_banco
     spring.datasource.username=seu_usuario
     spring.datasource.password=sua_senha
     spring.datasource.driver-class-name=org.postgresql.Driver
     spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
     spring.jpa.hibernate.ddl-auto=update
     ```

#### Parte 2: Modelagem de Dados

1. **Entidade `City`**
   - Crie uma entidade `City` para representar as cidades armazenadas no banco de dados:
     ```java
     @Entity
     @Table(name = "cities")
     public class City {
         @Id
         @GeneratedValue(strategy = GenerationType.IDENTITY)
         private Long id;
         
         private String name;
         private Double latitude;
         private Double longitude;
         
         // getters e setters
     }
     ```
   - Anote a entidade com `@Entity`, defina a chave primária (`@Id`) e adicione campos como nome, latitude e longitude.

#### Parte 3: Desenvolvimento da API REST

1. **Repositório de Cidades**
   - Crie um repositório `CityRepository` para acessar e manipular os dados das cidades no banco de dados:
     ```java
     import org.springframework.data.jpa.repository.JpaRepository;
     import org.springframework.data.jpa.repository.Query;
     import org.springframework.data.repository.query.Param;
     import java.util.List;
     
     public interface CityRepository extends JpaRepository<City, Long> {
         
         @Query("SELECT c FROM City c WHERE earth_distance(ll_to_earth(:lat, :lon), ll_to_earth(c.latitude, c.longitude)) <= :radius")
         List<City> findCitiesInRadius(@Param("lat") Double latitude, @Param("lon") Double longitude, @Param("radius") Double radius);
     }
     ```
   - Utilize uma consulta customizada com `@Query` para encontrar cidades dentro de um raio específico usando a função `earth_distance` do PostgreSQL.

2. **Controller da API**
   - Crie um controlador `CityController` para expor endpoints RESTful para buscar cidades dentro de um raio:
     ```java
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.web.bind.annotation.*;
     import java.util.List;
     
     @RestController
     @RequestMapping("/api/cities")
     public class CityController {
         
         @Autowired
         private CityRepository cityRepository;
         
         @GetMapping("/radius")
         public List<City> getCitiesInRadius(@RequestParam Double latitude, @RequestParam Double longitude, @RequestParam Double radius) {
             return cityRepository.findCitiesInRadius(latitude, longitude, radius);
         }
     }
     ```
   - Anote a classe com `@RestController` para indicar que é um controlador REST e defina o endpoint `/api/cities/radius` para receber os parâmetros de latitude, longitude e raio.

#### Parte 4: Implementação do Cálculo de Distância

1. **Função `earth_distance` no PostgreSQL**
   - Certifique-se de que o PostgreSQL tenha a extensão `earthdistance` instalada para utilizar a função `earth_distance` nas consultas.

   - Para instalar a extensão, execute o seguinte comando no PostgreSQL:
     ```sql
     CREATE EXTENSION IF NOT EXISTS cube;
     CREATE EXTENSION IF NOT EXISTS earthdistance;
     ```

2. **Testando a API**
   - Teste a API utilizando ferramentas como Postman ou através de scripts de cliente HTTP para verificar se as cidades dentro do raio são retornadas corretamente.

### Considerações Finais

Ao evoluir uma API de cidades para encontrar cidades dentro de um raio de distância usando Spring Boot e PostgreSQL, é importante configurar corretamente o ambiente de desenvolvimento, modelar os dados adequadamente, implementar consultas eficientes no banco de dados e testar minuciosamente para garantir que os requisitos sejam atendidos. Mantenha o código bem organizado, documentado e seguindo boas práticas de desenvolvimento para facilitar a manutenção e expansão futura do projeto.
