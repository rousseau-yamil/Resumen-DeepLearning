---

## 🐍 Archivo 2: `main.py`

```python
import os
from crewai import Agent, Task, Crew, Process, LLM
from IPython.display import Markdown

# --- CONFIGURACIÓN DEL MODELO ---
# Si usas OpenAI, descomenta la línea de abajo:
# os.environ["OPENAI_API_KEY"] = "TU_API_KEY"

# Si usas OLLAMA (Local y Gratis):
local_llm = LLM(
    model="ollama/llama3",
    base_url="http://localhost:11434"
)

## --- DEFINICIÓN DE AGENTES ---

planner = Agent(
    role="Content Planner",
    goal="Plan engaging and factually accurate content on {topic}",
    backstory="You're working on planning a blog article about the topic: {topic}. "
              "You collect information that helps the audience learn something "
              "and make informed decisions. Your work is the basis for "
              "the Content Writer to write an article on this topic.",
    allow_delegation=False,
    verbose=True,
    llm=local_llm
)

writer = Agent(
    role="Content Writer",
    goal="Write insightful and factually accurate opinion piece about the topic: {topic}",
    backstory="You're working on a writing a new opinion piece about the topic: {topic}. "
              "You base your writing on the work of the Content Planner, who provides an outline "
              "and relevant context. You follow the main objectives and direction provided. "
              "You acknowledge when your statements are opinions versus objective facts.",
    allow_delegation=False,
    verbose=True,
    llm=local_llm
)

editor = Agent(
    role="Editor",
    goal="Edit a given blog post to align with the writing style of the organization.",
    backstory="You are an editor who receives a blog post from the Content Writer. "
              "Your goal is to review the post to ensure it follows journalistic best practices, "
              "provides balanced viewpoints, and avoids major controversial topics.",
    allow_delegation=False,
    verbose=True,
    llm=local_llm
)

## --- DEFINICIÓN DE TAREAS ---

plan = Task(
    description=(
        "1. Prioritize the latest trends, key players, and noteworthy news on {topic}.\n"
        "2. Identify the target audience, considering their interests and pain points.\n"
        "3. Develop a detailed content outline including introduction, key points, and CTA.\n"
        "4. Include SEO keywords and relevant data or sources."
    ),
    expected_output="A comprehensive content plan document with an outline, audience analysis, SEO keywords, and resources.",
    agent=planner,
)

write = Task(
    description=(
        "1. Use the content plan to craft a compelling blog post on {topic}.\n"
        "2. Incorporate SEO keywords naturally.\n"
        "3. Ensure sections/subtitles are engaging.\n"
        "4. Structure with introduction, body, and conclusion.\n"
        "5. Proofread for grammar and brand voice alignment."
    ),
    expected_output="A well-written blog post in markdown format, ready for publication. Each section should have 2 or 3 paragraphs.",
    agent=writer,
)

edit = Task(
    description="Proofread the given blog post for grammatical errors and alignment with the brand's voice.",
    expected_output="A well-written blog post in markdown format, ready for publication. Each section should have 2 or 3 paragraphs.",
    agent=editor
)

## --- CREACIÓN Y EJECUCIÓN DEL EQUIPO ---

crew = Crew(
    agents=[planner, writer, editor],
    tasks=[plan, write, edit],
    verbose=True
)

print("### Iniciando el proceso de creación de contenido...")
result = crew.kickoff(inputs={"topic": "Artificial Intelligence"})

# Mostrar resultado
print("\n\n########################")
print("## RESULTADO FINAL ##")
print(result)


