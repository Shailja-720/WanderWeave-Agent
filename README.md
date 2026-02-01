# WanderWeave-Agent
Weave real-time data into romantic, budget-smart escapes
Inspiration Behind the Project
I drew inspiration from my own frustrating travel planning experiences—like spending hours juggling tabs for flights, hotels, and weather while forgetting my budget halfway through. Real-world tools like chatbots (e.g., basic ones on travel sites) felt too passive, spitting out generic advice without acting on constraints. The Gemini family of models, announced by Google in late 2023, promised a shift: from mere chat to true agents with tool-calling and reasoning. Papers like those on ReAct prompting and Google's Gemini technical reports fueled me, showing how multimodal models could handle real-time data. I wanted to prototype an "Intelligent Travel Agent" that turns vague queries like "romantic getaway under ₹5,000, 4-hour drive from Chandigarh" into executable plans.
Key Learnings
Building this deepened my grasp of agentic AI:
	Chatbot vs. Agent Dichotomy: Chatbots predict tokens autoregressively, limited by training data. Agents, powered by Gemini 1.5 Pro, decompose tasks via chain-of-thought (CoT) reasoning. For example, parsing constraints uses structured output:"Plan"=Decompose("Query")→ToolCall("APIs")→Validate("Constraints")
	Context retention via "Thought Signatures"—persistent memory embeddings that track user prefs across turns.
	Multimodality shines: Gemini processes user-uploaded images (e.g., "find similar vibe hotels") alongside text.
	Tool integration via Function Calling beats static info; real-time APIs make it dynamic.
I learned 80% of value comes from orchestration, not raw model power—handling errors, retries, and state is crucial.
How I Built the Project
I used LangChain for orchestration, Gemini API for the core model, and a mix of free/paid tools. Here's the step-by-step architecture:
	Frontend: Streamlit app for conversational UI, capturing voice/text inputs and user images.
	Agent Core (Gemini 1.5 Flash for speed):
	Parser: Extracts constraints into JSON: {"budget": 5000, "duration": "4h drive", "style": "romantic", "origin": "Chandigarh"}.
	Planner: CoT to break into steps: search destinations, query weather/hotels, build itinerary.
	Tools:
	Google Maps API for distances.
	Booking.com/Yatra APIs for real-time prices.
	OpenWeather for forecasts.
	Custom RAG over travel DBs for activities.
	State Management: Redis for session memory (prefs, partial plans).
	Safety Layer: Human-in-loop—pauses for approval on bookings via email/SMS.
	Deployment: Vercel for hosting, with rate-limiting.
Example flow for "romantic trip under 5k":
User: Romantic 4h drive from Chandigarh, <5k.
Agent: Parsed: Budget ₹5k, origin Chandigarh, style romantic.
Step 1: Nearby spots (Maps API) → Shimla (220km, ~4h).
Step 2: Weather (OpenWeather) → Clear, 15°C.
Step 3: Hotels (Booking API) → CozyTree Homestay, ₹3,200/night.
Itinerary: Day1: Drive+check-in; Day2: Mall Road stroll.
Confirm booking? [Yes/No]

Total build: 2 weeks, ~500 LOC Python.
Challenges Faced
	Context Loss: Long convos hit token limits (∼1M for Gemini 1.5 Pro, but still). Fixed with summarization: compress history to key facts.
	API Fragmentation: Inconsistent schemas (e.g., Yatra vs. Booking). Solution: Unified tool schema with Pydantic validation.
	Real-Time Reliability: APIs flake (downtime, rate limits). Added retries with exponential backoff:"Delay"=2^k×"base_delay",k=0,1,2,…
	Personalization Edge Cases: Vague prefs like "adventurous" → Used embeddings (Gemini multimodal) to match user photo vibes.
	Cost/Security: API calls add up (~₹0.5/query); bookings risky. Implemented mock mode for testing and Stripe for real ones.
	Hallucinations: Early versions invented prices. Mitigated with strict tool-grounding—no generation without API response.
Overcoming these made the agent 70% more reliable, handling 50+ test queries autonomously. It's a solid prototype—next up, voice mode with Gemini Live.
