using System;
using System.Collections.Generic;

namespace BuscadorDeVuelos
{
    // Clase que representa una conexión (arista) entre dos ciudades (nodos)
    class Arista
    {
        public string Destino { get; set; }
        public int Costo { get; set; }

        public Arista(string destino, int costo)
        {
            Destino = destino;
            Costo = costo;
        }
    }

    // Clase que representa el grafo de rutas aéreas
    class Grafo
    {
        private Dictionary<string, List<Arista>> adyacencia;

        public Grafo()
        {
            adyacencia = new Dictionary<string, List<Arista>>();
        }

        // Agregar ciudad (nodo)
        public void AgregarCiudad(string ciudad)
        {
            if (!adyacencia.ContainsKey(ciudad))
                adyacencia[ciudad] = new List<Arista>();
        }

        // Agregar conexión (arista dirigida con peso)
        public void AgregarRuta(string origen, string destino, int costo)
        {
            if (!adyacencia.ContainsKey(origen))
                AgregarCiudad(origen);
            if (!adyacencia.ContainsKey(destino))
                AgregarCiudad(destino);

            adyacencia[origen].Add(new Arista(destino, costo));
        }

        // Implementación del algoritmo de Dijkstra
        public void Dijkstra(string origen, string destino)
        {
            var distancias = new Dictionary<string, int>();
            var anteriores = new Dictionary<string, string>();
            var cola = new List<string>(adyacencia.Keys);

            foreach (var nodo in adyacencia.Keys)
                distancias[nodo] = int.MaxValue;

            distancias[origen] = 0;

            while (cola.Count > 0)
            {
                cola.Sort((x, y) => distancias[x].CompareTo(distancias[y]));
                var actual = cola[0];
                cola.RemoveAt(0);

                if (actual == destino)
                    break;

                foreach (var arista in adyacencia[actual])
                {
                    int nuevaDistancia = distancias[actual] + arista.Costo;
                    if (nuevaDistancia < distancias[arista.Destino])
                    {
                        distancias[arista.Destino] = nuevaDistancia;
                        anteriores[arista.Destino] = actual;
                    }

