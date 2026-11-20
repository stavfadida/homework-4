# Prompt and Change Record

This file records the assignment brief, the user instructions relevant to its implementation, and the changes actually made. The homework brief below is a structured transcription of the supplied PDF, not a claim that every sentence is verbatim.

## Original assignment brief

Source: AI_Agents_Homework_LangGraph_Topic_Chatbot.pdf  
Title: “Club Chatbot: Memory That Survives a Restart”

Build a topic-locked chatbot for the Curious Minds after-school clubs. The clubs are Astronomy, Cooking, Football, World Geography, and Python Programming. A child selects a club, and the bot answers only about the selected club. If the child changes clubs, the next answer follows the new topic, even if the prior conversation contains a question about another club. The chatbot must politely refuse off-topic questions and suggest switching clubs.

The conversation must survive a Colab runtime restart. Each conversation has its own thread_id. Users can start a new conversation, see saved conversations in a list, and reopen one with its messages and club restored. The conversation list must come from the database, not a Python variable. The assignment explicitly says not to build a weekly parent WhatsApp summary.

### Required graph and state

- Use LangGraph with state containing messages (using the add_messages reducer) and topic (without a reducer, so the new topic replaces the old one).
- Build exactly one chatbot node and exactly one LLM call for each user message.
- Build a fresh SystemMessage from the current topic on every call. Do not store that SystemMessage in the conversation history.
- Keep the graph structure START → chatbot → END.
- Use PostgresSaver connected to Supabase instead of InMemorySaver. Do not keep conversation history in a Python variable.
- Draw the graph in the notebook with draw_mermaid_png().

### Supabase and secrets

Create a Supabase project and use its Session pooler connection URI. Store SUPABASE_DB_URL and OPENAI_API_KEY in Google Colab Secrets. Do not paste either secret into notebook code, cell output, prompt.md, or GitHub. Create the PostgresSaver with autocommit=True, row_factory=dict_row, and prepare_threshold=0, then call setup() to create or migrate the checkpoint tables.

### Required interface

Build a Gradio Blocks interface with:

- A topic dropdown with at least four topics (the story provides five).
- A chat window and message box.
- A New conversation button that creates a new thread_id and clears the chat.
- A saved-conversations list read from Supabase. Each entry must have a readable label such as the topic and the opening question, not only a UUID.
- Selecting a saved conversation must load its messages and topic.
- Changing the topic must continue the same thread; from the next message onward, the bot follows only the selected topic.

The graph must also be exercised in a standalone notebook cell before the Gradio interface. The assignment’s four questions and topics are:

1. “How do I make pancakes?” — Cooking
2. “Who won the 2022 World Cup?” — Cooking
3. “Who won the 2022 World Cup?” — Football
4. “And how long do I fry each pancake?” — Football

After those turns, inspect graph.get_state(config).values. The sample in the PDF expects the saved topic to be Football and eight total messages (four human messages and four assistant messages).

### Submission and self-check

The assignment asks for a public GitHub repository containing prompt.md and the notebook, a Google Colab notebook link the lecturer can open, and a 4–6 sentence reflection. The reflection should describe actual observations about Supabase persistence/checkpoint tables, the off-topic test and any prompt adjustment, and what happens to earlier-topic messages after a topic switch.

The self-check asks that the notebook runs from a fresh Colab session after setting both secrets, that the graph has one chatbot node and a PostgresSaver, that old conversations and their topics can be restored after a restart, and that off-topic and earlier-topic questions are refused. The PDF describes an optional topic-guard node and a Delete conversation button as bonus work; those bonuses are not included in this implementation.

## Consolidated implementation prompt/specification

The following is the technical specification used to guide the notebook. It consolidates the supplied assignment PDF and the user's actual implementation instructions; it is not presented as a verbatim user message.

Build a simple Google Colab notebook for the Curious Minds club chatbot. Keep the source code in one notebook file during development. Use the five clubs Astronomy, Cooking, Football, World Geography, and Python Programming. Use LangGraph state with a messages channel using add_messages and a topic string that is replaced by the current topic on each input. Implement exactly one graph node named chatbot between START and END. In that node, build a new SystemMessage from the current topic, pass the system message and persisted conversation messages to the model, call the LLM once, and return only its assistant message. Never add the system message to the saved message state. Use MODEL_NAME = "openai:gpt-6-luna" in one place.

Use PostgresSaver connected to the Supabase Session pooler URL from Colab Secrets. Set autocommit=True, row_factory=dict_row, and prepare_threshold=0, call setup(), and keep the connection available while the UI is running. Never print or commit secrets. Use a UUID thread_id for new conversations. On each user message, pass only that new HumanMessage and the selected topic to graph.invoke using the same thread_id. A topic change continues the current thread and takes effect with the next user message.

Read saved conversations from checkpointer.list(None), not from a Python in-memory conversation registry. Keep the newest checkpoint for each thread and show a readable label containing its saved topic, first user question, and a short ID. When a saved item is selected, load it with graph.get_state and restore both its messages and topic. After a Colab runtime restart, the list must be rebuilt from Supabase so the user can reopen an older thread.

Use Gradio Blocks with a topic dropdown, a saved-conversations dropdown, a New conversation button, a chat window, and a message box. New conversation creates a new UUID thread and clears the visible chat without deleting prior database records. Sending a message invokes the graph once and refreshes the saved-conversation choices from Supabase. Selecting a saved conversation restores its state without calling the LLM.

Before the Gradio interface, include a standalone LangGraph test cell with these four exact questions and topics, in order: “How do I make pancakes?” (Cooking); “Who won the 2022 World Cup?” (Cooking); “Who won the 2022 World Cup?” (Football); “And how long do I fry each pancake?” (Football). Afterward inspect graph.get_state(config).values and check that the topic is Football and that there are eight messages. Print actual model replies only when the notebook is run; do not embed or claim expected answers as observed results.

Draw the graph with draw_mermaid_png(). Include a blank reflection template to be completed from actual Colab observations. Do not implement the optional topic-guard node or Delete conversation bonus. Do not invent test results or reflection. At the end, include prompt.md in the GitHub repository and commit and push project file changes as required by AGENTS.md.

## Additional user prompts and instructions

### 1. Initial planning request

“תקרא בבקשה את המטלה בקובץ PDF ששמורה בתיקייה הנוכחית ותבצע תכנון של קוד המטלה”

Effect: I read the supplied assignment PDF and returned an implementation plan. No code file was changed in response to that prompt.

### 2. Prompt-tracking instruction for AGENTS.md

“הוסף לקובץ AGENTS.md את ההוראה הבאה:

במהלך העבודה על הפרויקט, יש לתעד את כל הפרומפטים וההוראות שאני נותנת לך, כולל בקשות לתיקונים, שינויים ושיפורים בקוד.

בסיום פיתוח המטלה, צור קובץ בשם prompt.md שיכלול:

1. את הפרומפט המקורי שניתן לצורך ביצוע המטלה.
2. את כל הפרומפטים הנוספים שניתנו במהלך הפיתוח.
3. תיעוד של התיקונים והשינויים שבוצעו בעקבות כל פרומפט.
4. הסבר קצר על השינויים בין הגרסאות.

חשוב שהתיעוד ישקף את העבודה שבוצעה בפועל, ללא המצאת פרומפטים או פעולות שלא התרחשו.

קובץ prompt.md חייב להיות חלק ממאגר GitHub הסופי, בהתאם לדרישות המרצה.

לאחר עדכון AGENTS.md, בצע commit ו-push.”

Effect: AGENTS.md was updated and pushed in commit d7371ae before notebook implementation. This file, prompt.md, and the notebook are to be committed and pushed as requested.

### 3. Implementation requirements and approval

“התכנון נראה טוב. לפני תחילת המימוש, ודא שאתה מקפיד גם על הדרישות הבאות מתוך המטלה:

1. יש לכלול תא בדיקה עצמאי של LangGraph לפני ממשק Gradio, עם ארבע שאלות הבדיקה המדויקות שמופיעות במסמך המטלה, כולל בדיקת ה-State לאחר מכן.
2. רשימת השיחות השמורות חייבת להיטען ממסד הנתונים Supabase, ולא ממשתנה Python. לכל שיחה יש להציג שם קריא ולא רק thread_id.
3. יש לאפשר שחזור מלא של שיחה קודמת, כולל ההודעות והנושא הנבחר, גם לאחר הפעלה מחדש של סביבת Google Colab.
4. יש להקפיד על קריאה אחת בלבד למודל ה-LLM לכל הודעת משתמש, ועל צומת chatbot יחיד בגרף.
5. יש לתעד את הפרומפטים והשינויים שבוצעו במהלך הפיתוח, וליצור קובץ prompt.md בהתאם לדרישות המרצה.
6. אין להמציא תוצאות בדיקה או רפלקציה. את הרפלקציה נכתוב בהתאם לתוצאות הבדיקות שנבצע בפועל.
7. יש לשמור על פתרון פשוט, ברור ומותאם לרמת הקורס, ללא מימוש הבונוסים בשלב זה.

לאחר עדכון התכנון בהתאם להערות האלה, אפשר להתחיל במימוש המחברת.

בסיום, בצע commit ו-push בהתאם להוראות AGENTS.md, ודווח לי אילו קבצים נוצרו ומה עדיין דורש בדיקה ידנית ב-Google Colab.”

Effect: the plan was tightened before implementation. The notebook below follows these constraints. No model or Supabase test was run during notebook creation.

### 4. Approval to publish the repository

“כן, להפוך לציבורי”

Effect: after I explained that publishing would expose the committed files, the user explicitly approved making homework-4 public. I checked the committed project files for obvious API-key and credentialed-Postgres-URI patterns; neither pattern was found. The repository visibility was then changed from private to public.

## Changes actually made

- Created club_chatbot_supabase_gradio.ipynb as an 18-cell Google Colab notebook. It includes package setup, Colab Secrets instructions, a one-node graph, PostgresSaver setup, a graph diagram cell, the independent four-question test, database-backed conversation listing, restore/new-conversation callbacks, and the Gradio UI.
- The four assignment questions are present verbatim in the test cell. The test uses a fresh UUID-based thread per run so rerunning it does not append to an earlier test thread. It inspects and asserts the final state topic and message count.
- Saved conversation options are read via checkpointer.list(None); labels include the topic, first user question, and a short ID. Opening a conversation calls graph.get_state to restore messages and topic.
- The system message is rebuilt inside the chatbot node for each invocation and is not included in the message state. The graph contains one chatbot node, whose body calls the model once.
- The reflection section is an unfilled template. No test outputs or reflection results have been invented.
- After explicit user approval, changed the GitHub repository visibility to public to satisfy the assignment; the local untracked assignment PDF was not uploaded.

## Version notes

- Planning version: established the graph, state, checkpoint, persistence, and interface design from the PDF.
- Implementation version: added the Colab notebook and incorporated the seven user requirements above. During static review, repaired notebook JSON/string quoting and changed saved-conversation selection to a user-input event so programmatic dropdown updates do not trigger a reload.
- Final documentation version: this prompt.md records the original assignment brief, relevant user prompts, and actual implementation changes.

## Verification and remaining manual work

The notebook JSON was parsed and its cells were reviewed for the required questions, state checks, one-node graph, and database-backed conversation list. No notebook cell was executed, so this is not evidence that model calls, Supabase persistence, Gradio callbacks, or restart recovery have passed. No reflection results are recorded.

Manual Google Colab work remains:

1. Set OPENAI_API_KEY and SUPABASE_DB_URL in Colab Secrets and run the notebook from a fresh runtime.
2. Confirm the Supabase session-pooler connection and checkpoint tables work.
3. Run the standalone LangGraph test and record the actual answers and final state.
4. Run Gradio; check new conversations, topic switching in the same thread, database-backed labels, and reopening a conversation.
5. Restart the Colab runtime, rerun setup, reopen a saved conversation, and verify its messages and topic.
6. Write the 4–6 sentence reflection based on those observations.
7. Create and verify a Google Colab link the lecturer can open. The GitHub repository is already public. Do not publish the temporary Gradio link or secrets.

