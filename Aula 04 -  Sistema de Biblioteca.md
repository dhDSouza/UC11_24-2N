# 🚀 Aula 4 - Prática: Criando um Sistema de Biblioteca com Testes Automatizados

## 🎯 Objetivos da Aula

* Aplicar os conceitos das aulas anteriores em um projeto prático
* Desenvolver testes unitários para um sistema real
* Praticar com annotations JUnit e métodos de asserção
* Criar artefatos de teste completos
* Implementar testes parametrizados e de exceções

---

## 📚 Cenário: Sistema de Gerenciamento de Biblioteca

Vamos criar um sistema para gerenciar uma biblioteca digital. O sistema precisa controlar livros, empréstimos e usuários.

### 📋 Requisitos do Sistema (RF - Requisitos Funcionais)

**RF001** - Cadastrar livros no sistema  
**RF002** - Registrar usuários da biblioteca  
**RF003** - Realizar empréstimo de livros  
**RF004** - Devolver livros emprestados  
**RF005** - Consultar livros disponíveis  
**RF006** - Validar multas por atraso na devolução

---

## 🏗️ Estrutura do Projeto

### 📌 Classe Principal: Livro

```java
public class Livro {
    private String isbn;
    private String titulo;
    private String autor;
    private boolean disponivel;
    private int anoPublicacao;

    public Livro(String isbn, String titulo, String autor, int anoPublicacao) {
        this.isbn = isbn;
        this.titulo = titulo;
        this.autor = autor;
        this.anoPublicacao = anoPublicacao;
        this.disponivel = true;
    }

    // Getters e Setters
    public String getIsbn() { return isbn; }
    public String getTitulo() { return titulo; }
    public String getAutor() { return autor; }
    public boolean isDisponivel() { return disponivel; }
    public int getAnoPublicacao() { return anoPublicacao; }
    public void setDisponivel(boolean disponivel) { this.disponivel = disponivel; }
}
```

### 📌 Classe Principal: Usuario

```java
public class Usuario {
    private String id;
    private String nome;
    private String email;
    private boolean possuiMulta;

    public Usuario(String id, String nome, String email) {
        this.id = id;
        this.nome = nome;
        this.email = email;
        this.possuiMulta = false;
    }

    // Getters e Setters
    public String getId() { return id; }
    public String getNome() { return nome; }
    public String getEmail() { return email; }
    public boolean isPossuiMulta() { return possuiMulta; }
    public void setPossuiMulta(boolean possuiMulta) { this.possuiMulta = possuiMulta; }
}
```

### 📌 Classe Principal: Biblioteca

```java
import java.time.LocalDate;
import java.time.temporal.ChronoUnit;
import java.util.ArrayList;
import java.util.List;

public class Biblioteca {
    private List<Livro> livros;
    private List<Usuario> usuarios;
    private List<Emprestimo> emprestimos;

    public Biblioteca() {
        this.livros = new ArrayList<>();
        this.usuarios = new ArrayList<>();
        this.emprestimos = new ArrayList<>();
    }

    public void adicionarLivro(Livro livro) {
        if (livro == null) {
            throw new IllegalArgumentException("Livro não pode ser nulo");
        }
        livros.add(livro);
    }

    public void registrarUsuario(Usuario usuario) {
        if (usuario == null) {
            throw new IllegalArgumentException("Usuário não pode ser nulo");
        }
        usuarios.add(usuario);
    }

    public boolean emprestarLivro(String isbn, String usuarioId, LocalDate dataEmprestimo) {
        Livro livro = buscarLivroPorIsbn(isbn);
        Usuario usuario = buscarUsuarioPorId(usuarioId);

        if (livro == null || usuario == null) {
            return false;
        }

        if (!livro.isDisponivel() || usuario.isPossuiMulta()) {
            return false;
        }

        livro.setDisponivel(false);
        Emprestimo emprestimo = new Emprestimo(livro, usuario, dataEmprestimo);
        emprestimos.add(emprestimo);
        return true;
    }

    public double devolverLivro(String isbn, LocalDate dataDevolucao) {
        Emprestimo emprestimo = buscarEmprestimoAtivo(isbn);
        if (emprestimo == null) {
            return -1; // Empréstimo não encontrado
        }

        Livro livro = emprestimo.getLivro();
        livro.setDisponivel(true);

        // Calcular multa se houver atraso
        long diasAtraso = ChronoUnit.DAYS.between(emprestimo.getDataDevolucaoPrevista(), dataDevolucao);
        double multa = 0;
        
        if (diasAtraso > 0) {
            multa = diasAtraso * 2.0; // R$ 2,00 por dia de atraso
            emprestimo.getUsuario().setPossuiMulta(true);
        }

        emprestimo.setDataDevolucaoReal(dataDevolucao);
        return multa;
    }

    public List<Livro> listarLivrosDisponiveis() {
        return livros.stream()
                .filter(Livro::isDisponivel)
                .toList();
    }

    private Livro buscarLivroPorIsbn(String isbn) {
        return livros.stream()
                .filter(l -> l.getIsbn().equals(isbn))
                .findFirst()
                .orElse(null);
    }

    private Usuario buscarUsuarioPorId(String id) {
        return usuarios.stream()
                .filter(u -> u.getId().equals(id))
                .findFirst()
                .orElse(null);
    }

    private Emprestimo buscarEmprestimoAtivo(String isbn) {
        return emprestimos.stream()
                .filter(e -> e.getLivro().getIsbn().equals(isbn) && e.getDataDevolucaoReal() == null)
                .findFirst()
                .orElse(null);
    }
}
```

### 📌 Classe Auxiliar: Emprestimo

```java
import java.time.LocalDate;

public class Emprestimo {
    private Livro livro;
    private Usuario usuario;
    private LocalDate dataEmprestimo;
    private LocalDate dataDevolucaoPrevista;
    private LocalDate dataDevolucaoReal;

    public Emprestimo(Livro livro, Usuario usuario, LocalDate dataEmprestimo) {
        this.livro = livro;
        this.usuario = usuario;
        this.dataEmprestimo = dataEmprestimo;
        this.dataDevolucaoPrevista = dataEmprestimo.plusDays(14); // 14 dias para devolução
    }

    // Getters e Setters
    public Livro getLivro() { return livro; }
    public Usuario getUsuario() { return usuario; }
    public LocalDate getDataEmprestimo() { return dataEmprestimo; }
    public LocalDate getDataDevolucaoPrevista() { return dataDevolucaoPrevista; }
    public LocalDate getDataDevolucaoReal() { return dataDevolucaoReal; }
    public void setDataDevolucaoReal(LocalDate dataDevolucaoReal) { 
        this.dataDevolucaoReal = dataDevolucaoReal; 
    }
}
```

---

## 🧪 ATIVIDADE PRÁTICA: Implementar Testes Automatizados

### 📝 Parte 1: Criar os Casos de Teste

**Crie um documento com os seguintes casos de teste:**

| CTID | Descrição | Pré-condições | Passos | Resultado Esperado |
|------|-----------|---------------|--------|-------------------|
| CT001 | Cadastrar livro válido | - | 1. Criar livro com ISBN, título, autor<br>2. Adicionar à biblioteca | Livro cadastrado com sucesso |
| CT002 | Cadastrar livro nulo | - | 1. Tentar adicionar livro nulo | Deve lançar IllegalArgumentException |
| CT003 | Empréstimo bem-sucedido | Livro disponível, usuário sem multa | 1. Registrar empréstimo | Empréstimo realizado com sucesso |
| CT004 | Empréstimo com livro indisponível | Livro emprestado | 1. Tentar empréstimo | Empréstimo deve falhar |

### 📝 Parte 2: Implementar Testes Unitários

**Complete a classe de testes abaixo:**

```java
import org.junit.jupiter.api.*;
import java.time.LocalDate;
import static org.junit.jupiter.api.Assertions.*;

class BibliotecaTest {
    private Biblioteca biblioteca;
    private Livro livro1;
    private Livro livro2;
    private Usuario usuario1;
    private Usuario usuario2;

    @BeforeEach
    void setUp() {
        biblioteca = new Biblioteca();
        
        livro1 = new Livro("123456", "Dom Casmurro", "Machado de Assis", 1899);
        livro2 = new Livro("789012", "O Cortiço", "Aluísio Azevedo", 1890);
        
        usuario1 = new Usuario("U001", "João Silva", "joao@email.com");
        usuario2 = new Usuario("U002", "Maria Santos", "maria@email.com");
        
        // TODO: Adicionar livros e usuários à biblioteca
    }

    @Test
    @DisplayName("✅ Deve cadastrar livro com sucesso")
    void testCadastrarLivro() {
        // TODO: Implementar teste para cadastro de livro
        // Verificar se livro foi adicionado à biblioteca
    }

    @Test
    @DisplayName("❌ Deve lançar exceção ao cadastrar livro nulo")
    void testCadastrarLivroNulo() {
        // TODO: Implementar teste para livro nulo
        // Usar assertThrows para verificar exceção
    }

    @Test
    @DisplayName("✅ Deve realizar empréstimo com sucesso")
    void testEmprestarLivro() {
        // TODO: Implementar teste de empréstimo bem-sucedido
        // Verificar se livro ficou indisponível
    }

    @Test
    @DisplayName("❌ Não deve emprestar livro indisponível")
    void testEmprestarLivroIndisponivel() {
        // TODO: Implementar teste para livro já emprestado
    }

    @ParameterizedTest
    @DisplayName("💰 Deve calcular multa corretamente")
    @ValueSource(ints = {0, 5, 10}) // Dias de atraso
    void testCalcularMulta(int diasAtraso) {
        // TODO: Implementar teste parametrizado para cálculo de multa
        // Dica: usar LocalDate.now().plusDays(diasAtraso) para data de devolução
    }

    @Test
    @DisplayName("📚 Deve listar apenas livros disponíveis")
    void testListarLivrosDisponiveis() {
        // TODO: Implementar teste para listagem de livros disponíveis
        // Emprestar um livro e verificar se não aparece na lista
    }

    @Test
    @DisplayName("⏰ Deve bloquear empréstimo para usuário com multa")
    void testUsuarioComMulta() {
        // TODO: Implementar teste para usuário com multa
        // Configurar usuário com multa e tentar empréstimo
    }

    @AfterEach
    void tearDown() {
        // TODO: Limpar recursos se necessário
    }
}
```

### 📝 Parte 3: Criar Artefatos de Teste

**Desenvolva os seguintes documentos:**

#### 1. Plano de Teste para o Sistema de Biblioteca

---

# Plano de Teste - Sistema de Biblioteca

## 1. Objetivo
Validar o funcionamento do sistema de gerenciamento de biblioteca.

## 2. Escopo
- Testes unitários para todas as classes
- Testes de integração entre módulos
- Testes de funcionalidades críticas

## 3. Estratégia
- Desenvolvimento dirigido por testes (TDD)
- Cobertura mínima de 80% do código
- Testes automatizados com JUnit 5

## 4. Critérios de Aceitação
- Todos os testes devem passar
- Código deve estar livre de bugs críticos
- Performance: operações em menos de 100ms

---

#### 2. Matriz de Rastreabilidade

| Requisito | Casos de Teste | Status |
|-----------|----------------|--------|
| RF001 | CT001, CT002 | 🔄 |
| RF002 | CT003, CT007 | 🔄 |
| RF003 | CT003, CT004, CT007 | 🔄 |
| RF004 | CT005 | 🔄 |
| RF005 | CT006 | 🔄 |
| RF006 | CT005 | 🔄 |

### 📝 Parte 4: Desafio Avançado (Opcional)

**Implemente testes para cenários complexos:**

```java
@Test
@DisplayName("🎯 Teste integrado: fluxo completo de empréstimo e devolução")
void testFluxoCompleto() {
    // TODO: Implementar um teste que simule o fluxo completo:
    // 1. Cadastrar livro e usuário
    // 2. Realizar empréstimo
    // 3. Verificar indisponibilidade do livro
    // 4. Realizar devolução com atraso
    // 5. Verificar cálculo de multa
    // 6. Confirmar que livro está disponível novamente
}

@Test
@DisplayName("🔍 Teste de performance com múltiplos empréstimos")
void testPerformanceMultiplosEmprestimos() {
    // TODO: Implementar teste de performance
    // Usar assertTimeout para verificar tempo de execução
    // Simular 100 empréstimos simultâneos
}
```

---

## 📊 Critérios de Avaliação

| Critério | Peso | Descrição |
|----------|------|-----------|
| **Completude dos testes** | 40% | Todos os métodos principais testados |
| **Qualidade dos casos de teste** | 20% | Casos bem documentados e coesos |
| **Uso correto das annotations** | 20% | @BeforeEach, @Test, @ParameterizedTest, etc. |
| **Artefatos de teste** | 20% | Plano e matriz de rastreabilidade completos |

---

## 🎯 Resultado Esperado

Ao final desta atividade, os alunos devem ter:

✅ **Sistema de biblioteca funcional** com classes completas  
✅ **Suite de testes abrangente** cobrindo todos os cenários  
✅ **Artefatos de teste profissionais** (plano, casos, matriz)  
✅ **Domínio prático** das annotations e assertions do JUnit  
✅ **Experiência real** com desenvolvimento orientado a testes  

---

## 💡 Dicas 

1. **Comece pelos testes mais simples** (cadastro de livros)
2. **Use o método `assertThrows`** para testar exceções
3. **Experimente com `@ParameterizedTest`** para evitar código repetitivo
4. **Verifique sempre os estados antes e depois** das operações
5. **Documente bem os casos de teste** para facilitar a manutenção

**🚀 Boa sorte! Qualquer dúvida, estarei aqui tomando café ☕.**
