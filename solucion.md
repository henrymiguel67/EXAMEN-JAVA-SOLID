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
