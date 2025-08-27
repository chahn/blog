*August 2025 - Christian Hahn*

# **AI Agents, die in die Zukunft sehen können 🔮**

Okay, die Überschrift ist Clickbait, aber es stimmt. Wir arbeiten zusammen mit [Hugging Face](https://huggingface.co/), [Anthropic](https://www.anthropic.com/) und der Open Source Community daran, AI-Agents eine Superkraft zu verleihen: die Fähigkeit, in die Zukunft zu sehen. Das ist kein Trick, sondern ein technischer Game-Changer. Und so funktioniert's.

Bevor wir ins Detail gehen, klären wir kurz, was wir meinen, wenn wir von „Tools“ sprechen. Stell dir einen KI-Agenten wie einen digitalen Kollegen vor. Damit dieser Assistent Aufgaben in der digitalen Welt erledigen kann, benötigt er Werkzeuge. Das kann alles Mögliche sein: ein Tool, um weltweit aktuelle Wetterberichte abzufragen, der Zugriff auf deinen Google Kalender, um einen Termin zu planen, oder eine Verbindung zu deiner internen Kundendatenbank, um die neuesten Verkaufszahlen abzurufen. Die Fähigkeit, diese Tools clever und effizient zu nutzen, ist der Schlüssel zu einem wirklich leistungsfähigen Agenten.

#### **Der Aufstieg der Code-denkenden Agenten**

Im Jahr 2024 wurden mehrere [Forschungsarbeiten](#weiterführende-links), unter anderem von [Apple](https://machinelearning.apple.com/research/codeact), veröffentlicht, die eine grundlegende Erkenntnis lieferten: Agenten, die in Programmiersprache "denken" (Code Generation), sind Agenten, die in natürlicher Sprache schlussfolgern, deutlich überlegen. Dieser Ansatz, bekannt als "**CodeAct**", führt zu präziseren und zuverlässigeren Ergebnissen.

Als Reaktion auf diese Entwicklung hat Hugging Face [**SmolAgents**](https://huggingface.co/docs/smolagents/index) veröffentlicht, eine schlanke und leistungsstarke Open Source Bibliothek zur Entwicklung solcher KI-Agenten. Der zentrale Vorteil dieses Ansatzes liegt darin, dass der Agent seine Aufgaben und die Nutzung von externen Tools direkt in ausführbaren Code umsetzt. Anstatt in Prosa zu beschreiben, was es tun möchte, schreibt der Agent ein Programm, um sein Ziel zu erreichen.

#### **Das Problem: Die ineffiziente „Print-and-Inspect“-Vorgehensweise**

Bisher hatten diese CodeAct-Agenten jedoch eine große Ineffizienz: Um herauszufinden, welche Art von Informationen ein Tool zurückgibt, mussten sie es erst einmal aufrufen und sich das Ergebnis ansehen. Der Output des Tools landete direkt im "Kurzzeitgedächtnis" des Agenten, dem sogenannten **Context Window**.

Dieser Prozess führt zu mehreren Problemen:
1.  **Das Context Window ist begrenzt:** Wenn ein Tool eine große Datenmenge (z. B. eine lange Liste von Kundendaten) zurückgibt, kann dies das Gedächtnis des Agenten sprengen.
2.  **Verschwendung von Ressourcen:** Der Agent muss die Daten erst ausgeben und "ansehen", nur um ihre Struktur zu verstehen. Erst in einem zweiten Schritt kann er den Code schreiben, um diese Daten tatsächlich zu verarbeiten. Das kostet Zeit, Rechenleistung und bei token-basierten Modellen auch Geld.
3.  **Hohe "Mental Load":** Große Datenmengen im Context Window lenken den Agenten von seiner eigentlichen Aufgabe ab und verringern den verfügbaren Raum für komplexe Schlussfolgerungen.

Bisher war das der Alltag für KI-Agenten: 🤯

#### **Die Lösung: In die Zukunft sehen mit dem Output Schema**

Wir haben SmolAgents nun so erweitert, dass der Agent die Struktur der Tool-Antwort kennt, *bevor* er das Tool überhaupt aufruft. Er kann sozusagen in die Zukunft sehen.

Dies wird durch das sogenannte [Output Schema](https://huggingface.co/docs/smolagents/main/en/tutorials/tools#structured-output-and-output-schema-support) ermöglicht. Dem Agenten wird nicht nur mitgeteilt, *was* ein Tool tut, sondern auch, *welche Datenstruktur* es zurückgeben wird (z. B. eine Liste von Objekten mit den Feldern "Name" und "ID"). Mit diesem Wissen kann der Agent sofort den perfekten Code schreiben, um die Daten direkt und effizient zu verarbeiten. Der Output des Tools landet direkt in einer Variable im Code und muss nicht mehr das Context Window des LLMs belasten.

Der ineffiziente "Aufrufen → Anschauen → Verarbeiten"-Zyklus wird durch einen direkten und intelligenten Ansatz ersetzt.

#### **Relevanz im Zusammenspiel mit dem Model Context Protocol (MCP)**

Diese Entwicklung ist besonders im Zusammenspiel mit dem **Model Context Protocol (MCP)** relevant. Die neueste [Spezifikation](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema) von MCP unterstützt nun offiziell das Output Schema für Tools. Wir haben den MCP-Adapter entsprechend erweitert, um diese Information direkt an SmolAgents weiterzugeben. Dadurch können Agenten, die über MCP mit externen Tools kommunizieren, deren Antworten nun vorhersagen und optimal verarbeiten.

#### **Fazit: Schnellere und klügere Agenten**

Indem wir dem Agenten die Fähigkeit geben, die Zukunft von Tool-Aufrufen vorherzusehen, erzielen wir enorme Vorteile:
*   **Höhere Geschwindigkeit:** Der Agent arbeitet und "denkt" extrem viel schneller, da überflüssige Schritte entfallen.
*   **Effizientere Ressourcennutzung:** Das Kurzzeitgedächtnis (Context Window) bleibt schlank und wird nicht mit unnötigen Informationen überflutet.
*   **Besserer Fokus:** Da sich der Agent weniger merken muss, kann er sich besser auf das Lösen seiner eigentlichen Aufgaben konzentrieren.

Diese Weiterentwicklung macht KI-Agenten nicht nur leistungsfähiger, sondern bringt uns auch einen entscheidenden Schritt näher an wirklich autonome und effiziente Systeme.

---
#### Weiterführende Links
* [Executable Code Actions Elicit Better LLM Agents](https://arxiv.org/abs/2402.01030)
* [DynaSaur: Large Language Agents Beyond Predefined Actions](https://arxiv.org/abs/2411.01747)
* [If LLM Is the Wizard, Then Code Is the Wand: A Survey on How Code Empowers Large Language Models to Serve as Intelligent Agents](https://arxiv.org/abs/2401.00812)
* [Hugging Face smolagents: Structured Output and Output Schema Support](https://huggingface.co/docs/smolagents/main/en/tutorials/tools#structured-output-and-output-schema-support)
