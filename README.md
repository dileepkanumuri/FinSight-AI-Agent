

# Finsight AI Agent: Your Automated Financial Analyst

## The Big Picture: What Does It Do?

Imagine you have a company, and you want to understand how well it's doing financially compared to its competitors.  Normally, this would involve a *lot* of manual work:

1.  **Gathering Financial Data:** Finding income statements, balance sheets, etc., often in messy CSV files.
2.  **Analyzing the Data:** Calculating key ratios, identifying trends, and figuring out what the numbers actually *mean*.
3.  **Researching Competitors:**  Finding similar companies and gathering *their* financial data.
4.  **Comparing Performance:**  Putting all the data side-by-side and drawing conclusions about who's doing better and why.
5.  **Writing a Report:**  Summarizing all the findings in a clear, professional report.

Finsight AI Agent automates this entire process! You give it your company's financial data (in a CSV file), tell it who your competitors are, and it does the rest, generating a comprehensive financial comparison report.  It's like having a virtual financial analyst on demand.

## The Goal:  Why Build This?

The main goal is to save time and effort for anyone who needs to do financial analysis and competitive benchmarking.  This could be:

*   **Business Owners:**  To quickly understand their company's position in the market.
*   **Investors:** To evaluate potential investment opportunities.
*   **Financial Analysts:**  To automate the tedious parts of their job and focus on higher-level insights.
*   **Students/Researchers:** To easily conduct financial research projects.

By using AI, the process becomes faster, more consistent, and less prone to human error.

## How It Works: A Simple Explanation

Think of Finsight AI Agent as a team of specialized AI assistants working together. Each assistant has a specific job:

1.  **The Data Gatherer:**  Takes your CSV file and extracts the key financial information.  It understands the structure of financial statements.
2.  **The Analyst:**  Takes the raw financial data and performs calculations, identifies trends, and generates insights.  It's like a mini-expert in financial ratios.
3.  **The Researcher:**  Goes online (using the Tavily search engine) to find information about your competitors.  It's like a dedicated research assistant.
4.  **The Comparator:**  Compares your company's performance to the competitors, highlighting strengths and weaknesses.
5.  **The Report Writer:**  Takes all the analysis, research, and comparisons and writes a clear, professional report.
6.  **The Critic:** "Reviews" the report and provides suggestions for improvement, including identifying information gaps. This helps ensure the report is thorough.
7. **Research Critic:** "Researcher who provide suggestions for the improvement"

These "assistants" are actually different *nodes* in a workflow created using a framework called **LangGraph**.  LangGraph lets us connect these AI components together and define how they interact.  The workflow looks like this:

```
[Start] --> Gather Financials --> Analyze Data --> Research Competitors --> Compare Performance --(if needed)--> Collect Feedback --> Research Critique --> Compare Performance --> Write Report --> [End]
```

The agent can even go through multiple rounds of refinement.  If the "Critic" provides feedback, the "Researcher" can gather more information, and the "Comparator" and "Report Writer" can update the report.  This iterative process ensures a high-quality final product.

## Technical Details (For the Curious)

Let's get a *little* bit more technical, but I'll still keep it understandable.

*   **Language Model (LLM):**  The brains of the operation is `gpt-3.5-turbo` from OpenAI.  This is a powerful large language model that can understand and generate human-like text.  It's used for all the analysis, writing, and reasoning tasks.
*   **LangGraph:**  This is the framework that allows us to build the workflow (the "team of assistants").  It lets us define the different steps (nodes) and how they connect.  Think of it as the project manager for our AI team.
*   **LangChain:**  This is a library that provides tools and integrations for working with LLMs.  It's used to connect to the OpenAI API and to use the Tavily search tool.
*   **Tavily Search API:**  This is a search engine specifically designed for AI agents.  It allows the "Researcher" node to find relevant information about competitors online.
*   **State Management:**  LangGraph uses a `StateGraph` to keep track of all the information gathered at each step.  Think of it like a shared workspace for the AI assistants.  The `AgentState` (a Python `TypedDict`) defines the structure of this shared workspace (what information is stored).
*   **Checkpointer (Memory):** Finsight AI Agent utilizes `SqliteSaver` for memory, but it's configured to use `:memory:`. This means it creates a temporary, in-memory SQLite database.  This database stores the state of the conversation between steps, allowing the agent to "remember" what it's done.  **Important:** Because it's in-memory, the data is lost when the program ends.
*   **Streamlit:**  This is a Python library that makes it incredibly easy to build interactive web applications.  It's used to create the user interface where you upload your CSV file, enter competitors, and see the results.
*   **Pandas:** A powerful Python library for data analysis and manipulation. It's used to read and process the CSV data.

**Code Breakdown:**

1.  **`finance_agent.py`:**  This is the main file containing the LangGraph workflow.

    *   **Imports:**  Loads all the necessary libraries (LangChain, OpenAI, Tavily, etc.).
    *   **Environment Variables:**  Loads API keys from a `.env` file (you'll need to create this file and add your OpenAI and Tavily API keys).
    *   **`AgentState`:**  Defines the structure of the data that's passed between nodes (the shared workspace).
    *   **Prompts:**  Defines the instructions given to the LLM at each step (e.g., `GATHER_FINANCIALS_PROMPT`, `ANALYZE_DATA_PROMPT`).  These are crucial for guiding the AI's behavior.
    *   **Node Functions:**  Each function (e.g., `gather_financials_node`, `analyze_data_node`) represents a step in the workflow.  They take the current state, interact with the LLM or Tavily, and update the state.
    *   **`StateGraph`:**  The `builder` object creates the workflow graph, connecting the nodes.
    *   **`should_continue`:**  This function determines whether the agent should loop for another round of feedback or stop.
    *   **`graph.stream()`:**  This is where the magic happens!  It runs the workflow, passing the initial state and processing each step.
    * **Streamlit UI Code:** Sets up the web interface using Streamlit.  It handles file uploads, user input, and displaying the results.
        *   `st.title`, `st.text_input`, `st.text_area`, `st.file_uploader`, `st.button`:  These Streamlit functions create the UI elements.
        *   The `if st.button(...)` block runs the analysis when the user clicks the "Start Analysis" button.
        *   `graph.stream()` is called inside the Streamlit code to run the LangGraph workflow.
        *   `st.write()` displays the intermediate steps and the final report.

2. **`simple_agent.py`:** Demonstrates basic concept of Agent.
    * **`class Agent`:** A simplified example of an agent that can perform calculations and look up planet masses.  It shows the basic idea of a "thought-action-observation" loop.
    *   **`known_actions`:**  A dictionary mapping action names (like "calculate") to Python functions that perform those actions.
    *   **`query` and `query_interactive`:** Functions that demonstrate how to interact with the agent in a loop, handling its actions and observations.
    * **Regular Expression (`action_re`):** Used to parse the agent's output and identify actions.

3.  **`simple_agent_hum_in_loop.py`:** This shows a more advanced example of using LangGraph with tools and a human-in-the-loop.

    *   **`State`:**  Defines the state, which in this case only includes `messages`.
    *   **`TavilySearchResults`:**  Uses the Tavily search tool.
    *   **`model_with_tools`:**  Connects the LLM to the Tavily tool.
    *   **`bot`:**  The main chatbot function, which invokes the LLM.
    *   **`ToolNode` and `tools_condition`:**  Handles tool use and conditional branching in the graph.
    *   **`SqliteSaver` (Memory):**  Adds memory to the agent, allowing it to remember previous interactions.
    *   **Interactive Loop:**  Allows the user to chat with the agent, and the agent can use tools as needed.

4.  **`simple_agent_lngraph_tools.py`:** It's like `simple_agent_hum_in_loop.py`

5.  **`simple_agent_lngraph.py`:** This is the simplest example, showing a basic chatbot without tools.

    *   **`bot`:**  A simple function that invokes the LLM with the current messages.
    *   **No Tools:**  This example doesn't use any external tools.
    *   **No Conditional Edges:**  The graph is a straight line, with no branching.
    *   **Interactive Loop:**  Allows the user to chat with the agent.

## Getting Started: How to Run It

1.  **Clone the Repository:**
    ```bash
    git clone <your_repository_url>
    cd <your_repository_directory>
    ```

2.  **Install Dependencies:**
    ```bash
    pip install -r requirements.txt
    ```
    (You'll need to create a `requirements.txt` file listing the required packages: `langchain`, `langchain-openai`, `openai`, `python-dotenv`, `tavily-python`, `streamlit`, `pandas`, `langgraph`)

3.  **Set Up Environment Variables:**
    *   Create a file named `.env` in the project's root directory.
    *   Add your OpenAI API key and Tavily API key:
        ```
        OPENAI_API_KEY=your_openai_api_key
        TAVILY_API_KEY=your_tavily_api_key
        ```

4.  **Prepare Your Data:**
    *   Create a CSV file (e.g., `financials.csv`) containing your company's financial data. The exact format will depend on how you've structured your prompts, but it should include the key financial metrics you want to analyze.  The example code uses `pd.read_csv(StringIO(csv_file))` and `df.to_string(index=False)` to read the CSV data, so it expects a standard CSV format.

5.  **Run the Application:**
    ```bash
    streamlit run finance_agent.py
    ```
    This will start the Streamlit web application.  Open your web browser and go to the URL shown in the terminal (usually `http://localhost:8501`).

6.  **Use the Application:**
    *   Enter the task (e.g., "Analyze the financial performance of...").
    *   List your competitors (one per line).
    * Set maximum number or revision.
    *   Upload your CSV file.
    *   Click "Start Analysis".

## Results and Output

The Finsight AI Agent will process your data and generate a financial comparison report.  The report will be displayed in the Streamlit application. The report is generated iteratively.  The agent will first produce a draft, then collect feedback, research any areas needing improvement, and finally produce a revised report.  This process can repeat up to the `max_revisions` you set.

## Future Improvements

*   **More Sophisticated Financial Analysis:**  Add support for more advanced financial metrics and calculations (e.g., discounted cash flow analysis, valuation models).
*   **Multiple Data Sources:**  Allow the agent to gather data from sources other than CSV files (e.g., APIs, databases).
*   **Customizable Reporting:**  Give users more control over the report's format and content.
*   **User Authentication:**  Add user accounts and security features.
*   **Error Handling:**  Improve error handling and provide more informative messages to the user.
* **Persistent Storage:** Change the `SqliteSaver` to use a file-based database instead of `:memory:` to persist data between sessions.
* **More Tools:** Integrate with other tools, such as data visualization libraries (like matplotlib or Plotly) to create charts and graphs.

## Conclusion

Finsight AI Agent is a powerful tool for automating financial analysis and competitive benchmarking.  It demonstrates the potential of LLMs and agent frameworks like LangGraph to streamline complex tasks and provide valuable insights. By combining the strengths of AI with a user-friendly interface, Finsight AI Agent makes financial analysis accessible to a wider audience.


