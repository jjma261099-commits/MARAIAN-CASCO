using System;
using System.Collections.Generic;
using System.Globalization;

namespace RevistasCatalogo
{
    internal static class Program
    {
        // Catálogo en memoria
        private static readonly List<string> Catalogo = new()
        {
            "National Geographic",
            "Scientific American",
            "The Economist",
            "Nature",
            "Time",
            "Wired",
            "Forbes",
            "MIT Technology Review",
            "Harvard Business Review",
            "IEEE Spectrum"
        };

        private static void Main()
        {
            Console.OutputEncoding = System.Text.Encoding.UTF8;
            MostrarTitulo();

            bool salir = false;
            while (!salir)
            {
                MostrarMenu();
                Console.Write("Seleccione una opción: ");
                var opcion = Console.ReadLine();

                switch (opcion)
                {
                    case "1":
                        EjecutarBusqueda();
                        break;
                    case "2":
                        ListarCatalogo();
                        break;
                    case "3":
                        AgregarTitulo();
                        break;
                    case "4":
                        EliminarTitulo();
                        break;
                    case "0":
                        salir = true;
                        break;
                    default:
                        Console.WriteLine("⚠ Opción no válida. Intente de nuevo.");
                        break;
                }

                if (!salir)
                {
                    Console.WriteLine("\nPresione ENTER para continuar...");
                    Console.ReadLine();
                }
            }

            Console.WriteLine("¡Gracias por usar el catálogo de revistas!");
        }

        /* =========================
           LÓGICA DEL MENÚ
           ========================= */

        private static void MostrarTitulo()
        {
            Console.WriteLine("==============================================");
            Console.WriteLine("   Catálogo de Revistas - Búsqueda en C#");
            Console.WriteLine("==============================================\n");
        }

        private static void MostrarMenu()
        {
            Console.WriteLine("\nMenú principal");
            Console.WriteLine("1) Buscar un título");
            Console.WriteLine("2) Listar catálogo");
            Console.WriteLine("3) Agregar título");
            Console.WriteLine("4) Eliminar título");
            Console.WriteLine("0) Salir");
        }

        /* =========================
           ACCIONES
           ========================= */

        private static void EjecutarBusqueda()
        {
            Console.Write("Ingrese el título a buscar: ");
            string entrada = Console.ReadLine() ?? string.Empty;

            // Elegir algoritmo
            Console.WriteLine("Seleccione algoritmo de búsqueda:");
            Console.WriteLine("  a) Iterativo (lineal)");
            Console.WriteLine("  b) Recursivo (lineal)");
            Console.Write("Opción (a/b): ");
            string alg = (Console.ReadLine() ?? "").Trim().ToLowerInvariant();

            bool encontrado = alg switch
            {
                "a" => BusquedaIterativa(Catalogo, entrada),
                "b" => BusquedaRecursiva(Catalogo, entrada, 0),
                _ => BusquedaIterativa(Catalogo, entrada) // por defecto iterativa
            };

            Console.WriteLine(encontrado ? "✅ Encontrado" : "❌ No encontrado");
        }

        private static void ListarCatalogo()
        {
            if (Catalogo.Count == 0)
            {
                Console.WriteLine("El catálogo está vacío.");
                return;
            }

            Console.WriteLine("\nCatálogo actual:");
            for (int i = 0; i < Catalogo.Count; i++)
            {
                Console.WriteLine($" {i + 1}. {Catalogo[i]}");
            }
        }

        private static void AgregarTitulo()
        {
            Console.Write("Ingrese el nuevo título: ");
            string nuevo = (Console.ReadLine() ?? string.Empty).Trim();

            if (string.IsNullOrWhiteSpace(nuevo))
            {
                Console.WriteLine("⚠ El título no puede estar vacío.");
                return;
            }

            // Evitar duplicados (comparación normalizada)
            if (BusquedaIterativa(Catalogo, nuevo))
            {
                Console.WriteLine("⚠ El título ya existe en el catálogo.");
                return;
            }

            Catalogo.Add(nuevo);
            Console.WriteLine("✅ Título agregado correctamente.");
        }

        private static void EliminarTitulo()
        {
            Console.Write("Ingrese el título a eliminar: ");
            string target = Console.ReadLine() ?? string.Empty;

            int idx = IndexOfNormalizado(Catalogo, target);
            if (idx >= 0)
            {
                Catalogo.RemoveAt(idx);
                Console.WriteLine("✅ Título eliminado.");
            }
            else
            {
                Console.WriteLine("❌ No encontrado; no se realizó ninguna eliminación.");
            }
        }

        /* =========================
           ALGORITMOS DE BÚSQUEDA
           ========================= */

        /// <summary>
        /// Búsqueda lineal iterativa. O(n)
        /// </summary>
        private static bool BusquedaIterativa(List<string> lista, string objetivo)
        {
            string needle = Normalizar(objetivo);

            foreach (var item in lista)
            {
                if (Normalizar(item) == needle)
                    return true;
            }
            return false;
        }

        /// <summary>
        /// Búsqueda lineal recursiva. O(n)
        /// </summary>
        private static bool BusquedaRecursiva(List<string> lista, string objetivo, int indice)
        {
            if (indice >= lista.Count) return false;

            if (Normalizar(lista[indice]) == Normalizar(objetivo))
                return true;

            return BusquedaRecursiva(lista, objetivo, indice + 1);
        }

        /// <summary>
        /// Devuelve el índice del elemento (comparación normalizada) o -1 si no existe.
        /// </summary>
        private static int IndexOfNormalizado(List<string> lista, string objetivo)
        {
            string needle = Normalizar(objetivo);
            for (int i = 0; i < lista.Count; i++)
            {
                if (Normalizar(lista[i]) == needle)
                    return i;
            }
            return -1;
        }

        /// <summary>
        /// Normaliza cadenas para comparaciones: recorta, pasa a mayúsculas invariables y quita acentos.
        /// </summary>
        private static string Normalizar(string s)
        {
            if (s is null) return string.Empty;

            var trimmed = s.Trim().ToUpperInvariant();

            // Quitar acentos/diacríticos para comparaciones robustas
            var normalized = trimmed.Normalize(NormalizationForm.FormD);
            var chars = new List<char>(normalized.Length);
            foreach (var ch in normalized)
            {
                var uc = CharUnicodeInfo.GetUnicodeCategory(ch);
                if (uc != UnicodeCategory.NonSpacingMark)
                    chars.Add(ch);
            }
            return new string(chars.ToArray()).Normalize(NormalizationForm.FormC);
        }
    }
}
