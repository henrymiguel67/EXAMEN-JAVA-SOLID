# NO SE SIGUE EL PRINCIPIO (SRP)

# EXPLICACION: ya que en la clase generarreportes() se mezclas multiples responsabilidades como .registrarParticipantes ,calcularBonificaciones ,GestorCampeonato().

# Método principal para simular la ejecución del módulo

public static void main(String[] args) {
    GestorCampeonato gestor = new GestorCampeonato();
    gestor.registrarParticipantes();
    gestor.calcularBonificaciones();
    System.out.println("\n--- Generando Reportes ---");
    gestor.generarReportes("TEXTO");
    System.out.println("\n--- Generando más Reportes ---");
    gestor.generarReportes("HTML");
}


# solucion es separar cada una en diferentes clases 


class RegistroParticipantes {
    public void registrar(List<Equipo> equipos, List<Arbitro> arbitros) { ... }
}

class CalculadoraBonificaciones {
    public void calcular(List<Equipo> equipos) { ... }
}

interface Reporte {
    void generar(List<Equipo> equipos, List<Arbitro> arbitros);
}

# El Principio OCP Abierto-Cerrado  tenemos el problema en el metodo calcularBonificaciones que si le vamos a añadir mas metodos tendriamos que modificar el codigo y poner mas condicionales.


public void calcularBonificaciones() {
    System.out.println("\n--- Calculando Bonificaciones de Jugadores ---");
    for (Equipo equipo : equipos) {
    for (Jugador jugador : equipo.getJugadores()) {
    if (jugador.getPosicion().equals("Delantero")) {
    System.out.println("Calculando bonificación alta para Delantero: " + jugador.getNombre());}}}}

# solucion  le quitamos los condicionales ya que si queremos añadir un nuevo metodo sin tener que modificar el codigo usamos el polimorfismo.

abstract class Jugador {
    private String nombre;
    public Jugador(String nombre) { this.nombre = nombre; }
    public String getNombre() { return nombre; }
    public abstract void calcularBonificacion();
}


class Delantero extends Jugador {
    public Delantero(String nombre) { super(nombre); }
    public void calcularBonificacion() {
        System.out.println("Calculando bonificación alta para Delantero: " + getNombre());
    }
}


class Portero extends Jugador {
    public Portero(String nombre) { super(nombre); }
    public void calcularBonificacion() {
        System.out.println("Calculando bonificación estándar para Portero: " + getNombre());
    }
}


class Defensa extends Jugador {
    public Defensa(String nombre) { super(nombre); }
    public void calcularBonificacion() {
        System.out.println("Calculando bonificación base para Defensa: " + getNombre());
    }
}
# 3. LSP (Liskov Substitution Principle)

# Problema el `Jugador` es una clase con un campo `posicion`. Para que funcione correctamente hay que comprobar el valor de `posicion`, rompiendo el polimorfismo.

class Jugador {
    private String nombre;
    private String posicion; // "Portero", "Delantero", "Defensa"
    public Jugador(String nombre, String posicion) { this.nombre = nombre; this.posicion = posicion; }
    public String getNombre() { return nombre; }
    public String getPosicion() { return this.posicion; }
}

# Solución lo que hacemos es que el `Jugador` sea una clase abstracta y creamos subclases para cada tipo de jugador como: (`Delantero`, `Portero`, `Defensa`).  


   Jugador j = new Delantero("Juan Pérez");
j.calcularBonificacion();  // Funciona sin condicionales


# 4. ISP (Interface Segregation Principle)

# Problema en el método `generarReportes` maneja varios formatos (Texto, HTML) en una sola clase.  
Esto obliga a que la misma clase conozca lógica de varios reportes.


public void generarReportes(String formato) {
    if (formato.equals("TEXTO")) { ... }
    else if (formato.equals("HTML")) { ... }
}

# Solución lo que hacemos es crear nuevas interfaces específicas para cada tipo de reporte.


interface Reporte {
    void generar(List<Equipo> equipos, List<Arbitro> arbitros);
}

class ReporteTexto implements Reporte {
    public void generar(List<Equipo> equipos, List<Arbitro> arbitros) {
        StringBuilder sb = new StringBuilder();
        sb.append("--- Reporte del Campeonato (TEXTO) ---\n");
        sb.append("EQUIPOS:\n");
        for (Equipo e : equipos) sb.append("- ").append(e.getNombre()).append("\n");
        sb.append("ÁRBITROS:\n");
        for (Arbitro a : arbitros) sb.append("- ").append(a.getNombre()).append("\n");
        System.out.println(sb.toString());
    }
}

class ReporteHtml implements Reporte {
    public void generar(List<Equipo> equipos, List<Arbitro> arbitros) {
        StringBuilder html = new StringBuilder();
        html.append("<html><body>\n");
        html.append("  <h1>Reporte del Campeonato</h1>\n");
        html.append("  <h2>Equipos</h2>\n  <ul>\n");
        for (Equipo e : equipos) html.append("    <li>").append(e.getNombre()).append("</li>\n");
        html.append("  </ul>\n  <h2>Árbitros</h2>\n  <ul>\n");
        for (Arbitro a : arbitros) html.append("    <li>").append(a.getNombre()).append("</li>\n");
        html.append("  </ul>\n</body></html>");
        System.out.println(html.toString());
    }
}

# 5. DIP (Dependency Inversion Principle)

# Problema en el `GestorCampeonato` depende directamente de implementaciones concretas de reportes (Texto/HTML), lo que lo hace rígido.

public void generarReportes(String formato) {
    if (formato.equalsIgnoreCase("TEXTO")) { /* ... */ }
    else if (formato.equalsIgnoreCase("HTML")) { /* ... */ }
}


# Solución `GestorCampeonato` depende de la abstracción `Reporte`, y la implementación interfiere con el  tiempo de ejecución.


class GestorCampeonato {
    private RegistroParticipantes registro = new RegistroParticipantes();
    private CalculadoraBonificaciones calculadora = new CalculadoraBonificaciones();
    private List<Equipo> equipos = new ArrayList<>();
    private List<Arbitro> arbitros = new ArrayList<>();

    public void ejecutarCampeonato(Reporte reporte) {
        registro.registrar(equipos, arbitros);
        calculadora.calcular(equipos);
        reporte.generar(equipos, arbitros);
    }
}

# Código Final Completo
import java.util.ArrayList;
import java.util.List;

// --- Entidades ---
class Equipo {
    private String nombre;
    private List<Jugador> jugadores = new ArrayList<>();
    public Equipo(String nombre) { this.nombre = nombre; }
    public String getNombre() { return nombre; }
    public void agregarJugador(Jugador j) { this.jugadores.add(j); }
    public List<Jugador> getJugadores() { return this.jugadores; }
}

abstract class Jugador {
    private String nombre;
    public Jugador(String nombre) { this.nombre = nombre; }
    public String getNombre() { return nombre; }
    public abstract void calcularBonificacion();
}

class Delantero extends Jugador {
    public Delantero(String nombre) { super(nombre); }
    @Override
    public void calcularBonificacion() {
        System.out.println("Calculando bonificación alta para Delantero: " + getNombre());
    }
}

class Portero extends Jugador {
    public Portero(String nombre) { super(nombre); }
    @Override
    public void calcularBonificacion() {
        System.out.println("Calculando bonificación estándar para Portero: " + getNombre());
    }
}

class Defensa extends Jugador {
    public Defensa(String nombre) { super(nombre); }
    @Override
    public void calcularBonificacion() {
        System.out.println("Calculando bonificación base para Defensa: " + getNombre());
    }
}

class Arbitro {
    private final String nombre;
    public Arbitro(String nombre) { this.nombre = nombre; }
    public String getNombre() { return nombre; }
}

// --- SRP: Clases separadas ---
class RegistroParticipantes {
    public void registrar(List<Equipo> equipos, List<Arbitro> arbitros) {
        Equipo equipoA = new Equipo("Los Ganadores");
        equipoA.agregarJugador(new Delantero("Juan Pérez"));
        equipoA.agregarJugador(new Portero("Pedro Pan"));
        equipos.add(equipoA);
        System.out.println("Equipo 'Los Ganadores' registrado.");

        Equipo equipoB = new Equipo("Los Retadores");
        equipoB.agregarJugador(new Defensa("Alicia Smith"));
        equipos.add(equipoB);
        System.out.println("Equipo 'Los Retadores' registrado.");

        arbitros.add(new Arbitro("Miguel Díaz"));
        System.out.println("Árbitro 'Miguel Díaz' contratado.");
    }
}

class CalculadoraBonificaciones {
    public void calcular(List<Equipo> equipos) {
        System.out.println("\n--- Calculando Bonificaciones de Jugadores ---");
        for (Equipo equipo : equipos) {
            for (Jugador jugador : equipo.getJugadores()) {
                jugador.calcularBonificacion();
            }
        }
    }
}

// --- ISP: Interfaces de Reportes ---
interface Reporte {
    void generar(List<Equipo> equipos, List<Arbitro> arbitros);
}

class ReporteTexto implements Reporte {
    @Override
    public void generar(List<Equipo> equipos, List<Arbitro> arbitros) {
        StringBuilder contenido = new StringBuilder();
        contenido.append("--- Reporte del Campeonato (TEXTO) ---\n");
        contenido.append("EQUIPOS:\n");
        for (Equipo equipo : equipos) {
            contenido.append("- ").append(equipo.getNombre()).append("\n");
        }
        contenido.append("ÁRBITROS:\n");
        for (Arbitro arbitro : arbitros) {
            contenido.append("- ").append(arbitro.getNombre()).append("\n");
        }
        System.out.println(contenido.toString());
    }
}

class ReporteHtml implements Reporte {
    @Override
    public void generar(List<Equipo> equipos, List<Arbitro> arbitros) {
        StringBuilder html = new StringBuilder();
        html.append("<html><body>\n");
        html.append("<h1>Reporte del Campeonato</h1>\n");
        html.append("<h2>Equipos</h2>\n<ul>\n");
        for (Equipo equipo : equipos) {
            html.append("<li>").append(equipo.getNombre()).append("</li>\n");
        }
        html.append("</ul>\n<h2>Árbitros</h2>\n<ul>\n");
        for (Arbitro arbitro : arbitros) {
            html.append("<li>").append(arbitro.getNombre()).append("</li>\n");
        }
        html.append("</ul>\n</body></html>");
        System.out.println(html.toString());
    }
}

// --- DIP: Gestor depende de abstracciones ---
public class GestorCampeonato {
    private  RegistroParticipantes registro = new RegistroParticipantes();
    private  CalculadoraBonificaciones calculadora = new CalculadoraBonificaciones();
    private  List<Equipo> equipos = new ArrayList<>();
    private  List<Arbitro> arbitros = new ArrayList<>();

    public void ejecutarCampeonato(Reporte reporte) {
        registro.registrar(equipos, arbitros);
        calculadora.calcular(equipos);
        reporte.generar(equipos, arbitros);
    }

    public static void main(String[] args) {
        GestorCampeonato gestor = new GestorCampeonato();

        System.out.println("\n--- Generando Reporte TEXTO ---");
        gestor.ejecutarCampeonato(new ReporteTexto());

        System.out.println("\n--- Generando Reporte HTML ---");
        gestor.ejecutarCampeonato(new ReporteHtml());
    }
}