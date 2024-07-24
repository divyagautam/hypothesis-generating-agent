# hypothesis-generating-agent
Pathway and disease hypothesis generating agent, using KEGG and GO data 

## Setting up the environment
1. I used the `pyenv` to first setup a virtual environment
2. Using the requirements.txt installed the necessary packages in the environment
3. This is a development environment so lacks any tooling for testing or deployment
   
## Downloading Gene Association file
To run this code you will need to download the GAF file for Homo Sapiens from https://current.geneontology.org/products/pages/downloads.html

## Hypothesis generation notebook
To run the hypothesis generating task, you need to run the notebook `hypothesis_agent.ipynb`. It is self contained but needs the right enviorment to have been setup a-priori 

## Code Structure and Solution Architecture
### Graph Structure:
* Nodes:
    * Genes: Each gene in the KGML file is represented as a node with its primary symbol as the node's name, and additional attributes like aliases and KEGG link.
    * Compounds: Each compound in the KGML file is a node.
    * Pathways: Each pathway in the KGML file is a node.
* Edges:
    * Gene-Gene Relationships: The different types of relationships (PPrel, GErel, PCrel) between genes are captured as directed edges, with the relationship type and subtype as edge attributes.
    * Gene-Pathway Relationships: Genes are linked to the pathways they are part of using edges of type "in_pathway."
    * Gene-GO Annotations: Genes are connected to relevant GO terms (from the GAF file) with edges of type "has_go_annotation," along with additional attributes like GO aspect and evidence code.
How the Links are Created:
1. KGML Parsing (parse_kgml):
    * Extracts information about genes, compounds, pathways, and their relationships from the KGML file.
2. GAF Parsing (gaf_parser):
    * Extracts gene symbols and their GO annotations from the GAF file.
3. Graph Construction (create_knowledge_graph):
    * Creates nodes for each gene, compound, and pathway.
    * Adds directed edges between genes based on the relations in the KGML data.
    * Adds "in_pathway" edges to link genes to their corresponding pathways.
    * Matches genes from the GAF to their corresponding nodes in the graph using gene symbols.
    * Adds "has_go_annotation" edges between gene nodes and their GO terms, including GO aspect and qualifier information.

### Agent Design:
* Framework: I have used LangChain with OpenAI's GPT-4-o-mini model.
* Tools:
    * Knowledge Graph Query Tool: A custom tool was built to interact with the NetworkX knowledge graph. This tool will accept queries from the agent and return relevant information from the graph, such as:
        * Neighboring nodes (genes, compounds, pathways) of a given gene
        * Relationship types and subtypes between nodes
        * GO annotations associated with a gene
    * GPT-4-o-mini: It will receive the structured information from the knowledge graph tool and generate the hypotheses in natural language.
* Agent Workflow:
    * Receive Query: The agent executor takes a natural language query as input.
    * Parse Query: A simple rule-based parser extracts the key entities and relationships mentioned in the query.
    * Query Knowledge Graph: The knowledge graph tool will be called to retrieve relevant information from the graph based on the parsed query.
    * Generate Hypothesis: The evidence will be passed to the llm, which will generate the hypothesis and supporting explanation.
  
* Hypothesis Generation:
  The llm will use the following information to generate the hypotheses:
  * Query: The original query provided by the user.
  * Gene/Disease Information: Background information about the gene and disease from the KGML and GAF data (names, aliases, functions, etc.).
