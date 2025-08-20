# Insertion Sort für Strings (C#) – Schritt-für-Schritt mit Debug-Snippets

Dieses Markdown erklärt einen **anfängerfreundlichen** Insertion Sort für **Strings** – ohne `comesBefore`, ohne `goto`, ohne Extra-Methode und ohne `Math.Min`. Der Zeichenvergleich erfolgt direkt in der Schleife und ist ausführlich kommentiert.

---

## Vollständiger Code

```csharp
static void InsertionSort(string[] array)
{
    // Wir starten bei Index 1, denn links davon (Index 0) betrachten wir als bereits sortiert
    for (int currentIndex = 1; currentIndex < array.Length; currentIndex++)
    {
        string currentWord = array[currentIndex];   // Das Wort, das einsortiert werden soll
        int compareIndex = currentIndex - 1;        // Wir beginnen links daneben

        // Solange wir noch links Elemente haben
        while (compareIndex >= 0)
        {
            int letterIndex = 0;      // Zeichenposition im Wort
            int comparisonResult = 0; // -1 = currentWord kleiner, 0 = gleich, +1 = currentWord größer

            // Zeichenweise vergleichen, solange beide Wörter noch Zeichen haben
            while (letterIndex < currentWord.Length && letterIndex < array[compareIndex].Length)
            {
                if (currentWord[letterIndex] != array[compareIndex][letterIndex])
                {
                    // Unterschied gefunden → entscheiden
                    if (currentWord[letterIndex] < array[compareIndex][letterIndex])
                        comparisonResult = -1; // currentWord gehört weiter nach links
                    else
                        comparisonResult = 1;  // currentWord bleibt rechts

                    break; // Kein weiterer Zeichenvergleich nötig
                }
                letterIndex++; // Bis hier gleich → nächstes Zeichen prüfen
            }

            // Falls alle verglichenen Zeichen gleich waren → Länge entscheidet
            if (comparisonResult == 0 && currentWord.Length != array[compareIndex].Length)
            {
                if (currentWord.Length < array[compareIndex].Length)
                    comparisonResult = -1;   // kürzeres Wort ist lexikografisch kleiner
                else
                    comparisonResult = 1;    // längeres Wort ist größer
            }

            // Wenn currentWord kleiner ist → Element links nach rechts schieben und weiter nach links gehen
            if (comparisonResult < 0)
            {
                array[compareIndex + 1] = array[compareIndex];
                compareIndex--; // Wir rücken die Vergleichsposition nach links
            }
            else
            {
                // An dieser Stelle gehört currentWord hin
                break;
            }
        }

        // currentWord an der gefundenen Position ablegen
        array[compareIndex + 1] = currentWord;
    }
}
```

---

## Mini-Testprogramm

Mit diesem Snippet kannst du die Funktion schnell testen.

```csharp
static void Main()
{
    string[] words = { "Banane", "Apfel", "Birne", "Apfelsaft", "Apfel" };

    Console.WriteLine("Vorher : " + string.Join(", ", words));
    InsertionSort(words);
    Console.WriteLine("Nachher: " + string.Join(", ", words));
}
```

**Erwartete Ausgabe (alphabetisch):**

```text
Vorher : Banane, Apfel, Birne, Apfelsaft, Apfel
Nachher: Apfel, Apfel, Apfelsaft, Banane, Birne
```

---

## Debug-/Schritt-für-Schritt-Erklärung mit Code-Snippets

Wir sortieren folgendes Array:

```csharp
string[] words = { "Banane", "Apfel", "Birne" };
InsertionSort(words);
```

### Durchlauf 1 – `currentIndex = 1`

* `currentWord = "Apfel"`
* `compareIndex = 0` → vergleiche mit `"Banane"`

**Zeichenvergleich:**

```csharp
char c1 = currentWord[0];  // 'A' (65)
char c2 = array[0][0];     // 'B' (66)

// 'A' < 'B' → comparisonResult = -1
```

**Aktion:** `comparisonResult < 0` → schieben

```csharp
array[1] = array[0];      // ["Banane", "Banane", "Birne"]
compareIndex--;           // compareIndex = -1 → while endet
array[0] = currentWord;   // ["Apfel", "Banane", "Birne"]
```

---

### Durchlauf 2 – `currentIndex = 2`

* `currentWord = "Birne"`
* `compareIndex = 1` → vergleiche mit `"Banane"`

**Zeichenvergleich:**

```csharp
// Index 0: 'B' == 'B' → weiter
// Index 1: 'i' (105) vs 'a' (97) → unterschiedlich
int comparisonResult = (105 < 97) ? -1 : 1; // → 1 (currentWord größer)
```

**Aktion:** `comparisonResult >= 0` → richtige Stelle gefunden, NICHT schieben

```csharp
array[compareIndex + 1] = currentWord; // bleibt an Position 2
// Array bleibt: ["Apfel", "Banane", "Birne"]
```

---

## Eckfälle & Hinweise

* **Gleiche Präfixe**: Wenn zwei Wörter bis zum Ende gleich sind (z. B. `"Apfel"` und `"Apfelsaft"`), entscheidet die **Länge**. Das kürzere Wort kommt zuerst.
* **Stabile Sortierung**: Diese Implementierung ist stabil in dem Sinne, dass gleiche Elemente ihre relative Reihenfolge behalten.
* **Groß-/Kleinschreibung**: Standardmäßig sortiert C# mit `'A' < 'a'`. Wenn du **case-insensitive** willst, kannst du vor dem Vergleich lokal je Zeichen `char.ToLower(...)` verwenden:

  ```csharp
  char left = char.ToLower(currentWord[letterIndex]);
  char right = char.ToLower(array[compareIndex][letterIndex]);
  if (left != right)
      comparisonResult = (left < right) ? -1 : 1;
  ```

  (Achtung: Das ändert die **Sortierreihenfolge**, ist aber für viele Anwendungsfälle erwünscht.)

---

## Warum ist was wo?

* **`currentIndex = 1`**: Bei `0` gibt es kein linkes Element – der Bereich `0..currentIndex-1` gilt als bereits sortiert.
* **`currentWord` merken**: Damit wir beim Schieben von Elementen nicht den Wert verlieren, den wir einsortieren wollen.
* **Zeichenweise Vergleich im `while`**: So vermeiden wir eine Extra-Methode und `Math.Min`. Wir prüfen explizit, dass beide Wörter an `letterIndex` noch ein Zeichen haben.
* **Längen-Check nach dem Zeichenvergleich**: Nur wenn bis dahin alle Zeichen gleich waren, entscheidet die Länge.
* **Schieben statt Tauschen**: Insertion Sort schiebt alle größeren Elemente um 1 nach rechts, bis die korrekte Position erreicht ist.

---

## Kompakte Variante (gleiches Verhalten, weniger Kommentare)

```csharp
static void InsertionSort(string[] array)
{
    for (int currentIndex = 1; currentIndex < array.Length; currentIndex++)
    {
        string currentWord = array[currentIndex];
        int compareIndex = currentIndex - 1;

        while (compareIndex >= 0)
        {
            int letterIndex = 0;
            int comparisonResult = 0;

            while (letterIndex < currentWord.Length && letterIndex < array[compareIndex].Length)
            {
                if (currentWord[letterIndex] != array[compareIndex][letterIndex])
                {
                    comparisonResult = (currentWord[letterIndex] < array[compareIndex][letterIndex]) ? -1 : 1;
                    break;
                }
                letterIndex++;
            }

            if (comparisonResult == 0 && currentWord.Length != array[compareIndex].Length)
                comparisonResult = (currentWord.Length < array[compareIndex].Length) ? -1 : 1;

            if (comparisonResult < 0)
            {
                array[compareIndex + 1] = array[compareIndex];
                compareIndex--;
            }
            else
            {
                break;
            }
        }

        array[compareIndex + 1] = currentWord;
    }
}
```

---

**Viel Erfolg beim Lernen & Debuggen!**
