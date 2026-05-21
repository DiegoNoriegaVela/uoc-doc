import javax.naming.Context;
import javax.naming.NamingException;
import javax.naming.directory.DirContext;
import javax.naming.directory.InitialDirContext;
import java.util.Hashtable;

public class LdapTest {

    public static void main(String[] args) {
        // ── Parámetros ────────────────────────────────────────────
        String ldapUrl    = "ldap://UATPER.BNS:389";
        String domain     = "UATPER.BNS";
        String username   = args.length > 0 ? args[0] : "tu_usuario";
        String password   = args.length > 1 ? args[1] : "tu_password";
        // Active Directory espera: usuario@dominio  ó  DOMINIO\\usuario
        String principal  = username + "@" + domain;
        // ─────────────────────────────────────────────────────────

        Hashtable<String, String> env = new Hashtable<>();
        env.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
        env.put(Context.PROVIDER_URL,            ldapUrl);
        env.put(Context.SECURITY_AUTHENTICATION, "simple");
        env.put(Context.SECURITY_PRINCIPAL,      principal);
        env.put(Context.SECURITY_CREDENTIALS,    password);
        // Timeout de conexión y lectura (ms)
        env.put("com.sun.jndi.ldap.connect.timeout", "5000");
        env.put("com.sun.jndi.ldap.read.timeout",    "5000");

        System.out.println("Intentando conectar a: " + ldapUrl);
        System.out.println("Usuario (principal): "   + principal);

        try {
            DirContext ctx = new InitialDirContext(env);
            System.out.println("✔ Autenticación LDAP exitosa.");
            ctx.close();
        } catch (NamingException e) {
            System.err.println("✘ Fallo de autenticación LDAP.");
            System.err.println("Causa: " + e.getMessage());
            // Código de error LDAP viene en el mensaje, p.ej. [LDAP: error code 49]
        }
    }
}

# Compilar
javac LdapTest.java

# Ejecutar pasando usuario y password como argumentos
java LdapTest miusuario mipassword

#!/bin/bash
echo "=== Buscando Java en el sistema ==="

# 1. Comando directo
echo -e "\n[1] which / type:"
which java 2>/dev/null && echo "  -> $(java -version 2>&1 | head -1)" || echo "  No en PATH"

# 2. Rutas comunes
echo -e "\n[2] Rutas comunes:"
for dir in /usr/bin/java /usr/local/bin/java /opt/java /opt/jdk* /opt/jre* \
           /usr/lib/jvm /usr/java /usr/local/java; do
    [ -e "$dir" ] && echo "  Encontrado: $dir"
done

# 3. Buscar binario en filesystem
echo -e "\n[3] find en /usr y /opt (puede tardar):"
find /usr /opt -name "java" -type f 2>/dev/null | while read p; do
    echo "  $p  -> $($p -version 2>&1 | head -1)"
done

# 4. Alternativas registradas (Red Hat / CentOS)
echo -e "\n[4] alternatives:"
alternatives --list 2>/dev/null | grep java || echo "  Sin registros"

# 5. RPMs instalados
echo -e "\n[5] RPMs de Java:"
rpm -qa | grep -i java 2>/dev/null || echo "  Ninguno"

# 6. JAVA_HOME en el entorno
echo -e "\n[6] JAVA_HOME: ${JAVA_HOME:-'No definido'}"

echo -e "\n=== Fin ==="

chmod +x find_java.sh
./find_java.sh