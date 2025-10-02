```python
# -*- coding: utf-8 -*-
"""
Symbolische Herleitung des Gaußschen Integrals mit SymPy
Architektur-Basis: Oberste Direktive Hyper Python V5
Mission: Die mathematische Eleganz der Herleitung in Python-Code nachbilden.
Hexen-Modus: "Wir heben das Problem in eine höhere Dimension,
             um seine wahre, einfache Form in einem anderen Licht zu erkennen."
"""

import sympy as sp
import numpy as np
from scipy import integrate

# --- Phase 1: Definition der Symbole und des Integrals ---
#
# Wir definieren alle notwendigen Symbole. In der Mathematik sind dies unsere Variablen.
# In unserem Code sind sie die Bausteine unserer symbolischen Berechnungen.
x, y, r = sp.symbols('x y r', real=True, positive=True)
theta = sp.symbols('theta', real=True)

print("--- Symbolische Herleitung des Gaußschen Integrals ---")
print("Ziel: Berechne I = ∫[0, ∞] exp(-x^2) dx\n")


# Das ursprüngliche Integral, das wir lösen wollen.
# Wir definieren es symbolisch, anstatt es direkt auszuwerten.
I = sp.Integral(sp.exp(-x**2), (x, 0, sp.oo))

print(f"Schritt 1: Das ursprüngliche Integral I =")
sp.pprint(I)
print("\n")


# --- Phase 2: Der Trick - Quadrieren und Wechsel in 2D ---
#
# Protokoll 4 (Oberste Direktive): Wir verkennen die Realität nicht, dass dieses
# Integral keine einfache Stammfunktion hat. Also ändern wir die Perspektive.
# Wir betrachten I², um es in ein Doppelintegral zu überführen.
# Hexe: "Ein einzelner Faden ist schwer zu messen. Weben wir ein Tuch daraus."

# Da die Integrationsvariable beliebig ist, können wir I² schreiben als:
# I² = (∫ exp(-x²) dx) * (∫ exp(-y²) dy) = ∫∫ exp(-(x²+y²)) dx dy
# Die Grenzen ändern sich von [0, ∞] zu [-∞, ∞], was einen Faktor 1/4 einführt.
# I_full = ∫[-∞, ∞] exp(-x^2) dx = 2 * I.
# I_full² = 4 * I²

I_full_sq = sp.Integral(sp.exp(-(x**2 + y**2)), (x, -sp.oo, sp.oo), (y, -sp.oo, sp.oo))

print("Schritt 2: Quadrieren des Integrals (über den gesamten Bereich) zu I_full² =")
sp.pprint(I_full_sq)
print("\n")


# --- Phase 3: Transformation in Polarkoordinaten ---
#
# Der entscheidende Schritt. In kartesischen Koordinaten ist das Integral schwer.
# In Polarkoordinaten offenbart es seine Einfachheit.
# Hexe: "Wir lassen die Koordinaten tanzen, bis sie uns die Lösung zuflüstern."
#
# Transformation:
# x = r * cos(θ)
# y = r * sin(θ)
# x² + y² = r²
# dx dy = r dr dθ (das 'r' ist die Jacobi-Determinante)
#
# Grenzen:
# r geht von 0 bis ∞
# θ geht von 0 bis 2π

polar_integral = sp.Integral(sp.exp(-r**2) * r, (r, 0, sp.oo), (theta, 0, 2*sp.pi))

print("Schritt 3: Transformation in Polarkoordinaten:")
sp.pprint(polar_integral)
print("\n")


# --- Phase 4: Symbolische Lösung ---
#
# Jetzt, in der neuen Form, können wir das Integral leicht lösen.
# SymPy führt die Integration für uns durch.
result_I_full_sq = polar_integral.doit()

print(f"Schritt 4: Lösen des Integrals in Polarkoordinaten...")
print(f"Ergebnis für I_full² = {result_I_full_sq}\n")

# Wir wissen: I_full² = 4 * I²
# Also: I² = result_I_full_sq / 4
result_I_sq = result_I_full_sq / 4

# Und I ist die Wurzel daraus.
result_I = sp.sqrt(result_I_sq)

print(f"Schritt 5: Rückrechnung auf das ursprüngliche Integral I")
print(f"I² = I_full² / 4 = {result_I_sq}")
print(f"I = sqrt(I²) = {result_I}")

print("\n--- Symbolische Herleitung abgeschlossen ---\n")


# --- Phase 5: Numerische Verifikation (Pragmatismus-Check) ---
#
# Protokoll 3 (Betriebssystem): Wir untermauern die theoretische Eleganz
# mit einem harten, numerischen Beweis. Wir verwenden SciPy, das Arbeitspferd
# für numerische Integration in Python.

print("--- Numerische Verifikation mit SciPy ---")

# Die zu integrierende Funktion
f = lambda x: np.exp(-x**2)

# Numerische Integration von 0 bis unendlich
# quad gibt das Ergebnis und eine Schätzung des Fehlers zurück
numerical_result, error = integrate.quad(f, 0, np.inf)

print(f"Numerisches Ergebnis von SciPy: {numerical_result}")
print(f"Symbolisches Ergebnis von SymPy: {result_I.evalf()}") # .evalf() zur numerischen Auswertung
print(f"Geschätzter Fehler der numerischen Methode: {error}")

# Vergleich
assert np.isclose(numerical_result, result_I.evalf()), "Numerisches und symbolisches Ergebnis stimmen nicht überein!"
print("\nVergleich erfolgreich: Die Ergebnisse stimmen überein.")
print("Mission erfüllt. Die Eleganz der Mathematik wurde erfolgreich in Python-Code gespiegelt.")

# Zusätzliche Visualisierung
import matplotlib.pyplot as plt

def plot_gaussian():
    x_vals = np.linspace(-3, 3, 1000)
    y_vals = np.exp(-x_vals**2)
    plt.figure(figsize=(10, 6))
    plt.plot(x_vals, y_vals, 'b-', linewidth=2, label='$e^{-x^2}$')
    plt.fill_between(x_vals[x_vals>=0], y_vals[x_vals>=0], alpha=0.3, label='Integralbereich')
    plt.title('Gaußsche Glockenkurve')
    plt.legend()
    plt.grid(True)
    plt.show()

```
