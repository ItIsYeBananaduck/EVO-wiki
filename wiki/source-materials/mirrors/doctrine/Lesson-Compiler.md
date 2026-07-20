# Lesson Compiler

## Input sources
- teacher PDF lesson plan
- worksheet scans/photos
- slide decks (PDF export)
- lecture transcript (optional)
- teacher annotations (optional)

## Output
- LessonPack JSON (versioned)
- MethodProfile (teacher style)
- Safe practice bank
- Template refs for interactive activities

## Compiler stages
1. Extract structure
   - headings, topics, objectives
2. Extract concepts
   - definitions, vocab, key rules
3. Extract examples
   - worked examples, sample problems
4. Extract practice
   - questions, worksheets, prompts
5. Build assessments
   - quiz items + rubrics
6. Apply teacher MethodProfile
   - sequencing + tone + pacing constraints
7. Apply Kids safety constraints
   - bounded topics + no open chat + template-only games
8. Emit versioned LessonPack