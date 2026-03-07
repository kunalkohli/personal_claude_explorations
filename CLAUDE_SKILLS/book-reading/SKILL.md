---
name: book-reading
description: Book reading session - discuss chapters, capture learnings, and maintain a running book summary in Obsidian
---

# Book Reading Session

Discuss a book chapter the user read, capture key learnings and thoughts, then create or update a running markdown summary in Obsidian with a TLDR and detailed chapter notes.

## Configuration

```
Vault: <YOUR_OBSIDIAN_VAULT_PATH>
Book notes: {vault}/Book Reading/{Book Title}.md
```

> **Setup:** Replace `<YOUR_OBSIDIAN_VAULT_PATH>` with the absolute path to your Obsidian vault.

## Workflow

### Step 1: Identify the Book

Ask the user which book they read tonight:

```
What book did you read tonight?
```

Once the user names the book, check if a markdown file already exists for it at:
`{vault}/Book Reading/{Book Title}.md`

- Use the book title as the filename (e.g., `Thinking Fast and Slow.md`, `The Pragmatic Programmer.md`)
- Normalize the title for the filename: title case, spaces preserved, no special characters that break filesystems (replace `:` `/` `\` with `-`)

If the file exists, read it to understand what chapters have already been covered, so you can pick up where they left off and avoid asking about things already discussed.

### Step 2: Identify the Chapter

Ask what chapter or section they read:

```
What chapter did you read? (number, name, or both)
```

If the book file already exists and has previous chapters logged, you can suggest:
```
Last time we covered Chapter 5 - "The Anchoring Effect". Did you read Chapter 6 tonight?
```

### Step 3: Discuss the Reading

This is the core of the session. Have a genuine, engaging conversation about what they read. Go through these prompts **one at a time** - do NOT batch them. Wait for a response before asking the next.

**Start with an open-ended prompt:**
```
What stood out to you most in this chapter?
```

**Then explore further based on their response. Pick 2-3 of these depending on what feels natural:**

- What's the main argument or idea the author is making?
- Did anything surprise you or challenge what you previously thought?
- How does this connect to your work or life?
- Was there a specific example, story, or analogy that stuck with you?
- Did you disagree with anything the author said?
- How does this chapter build on what came before?

**Keep the discussion natural** - respond to what they say, share relevant connections, push back gently if something is worth examining deeper. This should feel like a conversation with a well-read friend, not an interrogation.

**When the conversation naturally winds down** (user gives shorter answers, says "that's about it", etc.), move to Step 4.

### Step 4: Summarize and Confirm

Synthesize the discussion into:

1. **TLDR bullet points** (1-2 points) - the highest-level takeaways from this chapter that would make sense even to someone who hasn't read the book
2. **Chapter notes** - a richer summary capturing key ideas, the user's thoughts, memorable quotes or examples, and any personal connections they made

Present both in the terminal for review:

```
Here's what I captured from tonight's session:

**TLDR additions:**
- [High-level takeaway 1]
- [High-level takeaway 2]

**Chapter X: [Title]** (YYYY-MM-DD)
[Detailed notes paragraph(s)]

Look good? (yes / make changes)
```

### Step 5: Write to Obsidian

**IMPORTANT**: Fetch today's date before writing:
```bash
date +%Y-%m-%d
```

#### If the file does NOT exist - create it:

```markdown
# {Book Title}
**Author:** {Author Name}
**Started:** YYYY-MM-DD

## TLDR
- [Takeaway 1 from tonight]
- [Takeaway 2 from tonight]

---

## Chapter Notes

### Chapter {N}: {Chapter Title} (YYYY-MM-DD)

{Detailed notes from the discussion}
```

#### If the file EXISTS - update it:

1. **Append new TLDR bullets** to the existing `## TLDR` section (add new lines after the last bullet under TLDR, before the `---`)
2. **Append new chapter notes** at the end of the `## Chapter Notes` section

The TLDR section grows over time into a complete high-level summary of the entire book. The chapter notes section is the detailed chronological record.

## File Naming

- Use the book's title as-is for readability: `Thinking Fast and Slow.md`
- Replace filesystem-unsafe characters: `:` `/` `\` `?` `*` `"` `<` `>` `|` become `-`
- Keep spaces - Obsidian handles them fine
- Example: `How to Win Friends & Influence People.md`
- Example: `Sapiens - A Brief History of Humankind.md` (colon replaced with dash)

## TLDR Guidelines

The TLDR section should be scannable and self-contained. Each bullet should:
- Be understandable without having read the chapter
- Capture an insight, not just a topic (e.g., "We have two thinking systems - fast intuitive and slow deliberate - and most errors come from over-relying on the fast one" not "Chapter 1 talks about two systems")
- Be 1-2 sentences max
- Accumulate over time so by the end of the book, the TLDR is a standalone summary

## Error Handling

- If vault path doesn't exist: Error and stop
- If the book file can't be read: Warn user, offer to create fresh
- If user isn't sure of chapter number: That's fine - use a descriptive title instead (e.g., "The part about cognitive biases")
- If user wants to log multiple chapters in one session: Handle each one, adding TLDR bullets and notes for each

## Example Session

```
User: /book-reading

Claude: What book did you read tonight?

User: Thinking Fast and Slow

Claude: [Checks vault - no existing file found]
       What chapter did you read?

User: Chapter 1 - The Characters of the Story

Claude: What stood out to you most in this chapter?

User: The idea that we have two systems - System 1 is fast and intuitive,
      System 2 is slow and deliberate. I never really thought about how
      much of my thinking is System 1.

Claude: That's the foundation Kahneman builds everything on. What's
       interesting is most people assume they're operating in System 2
       most of the time, but research shows the opposite. Did he give
       any examples that made you realize how much you rely on System 1?

User: Yeah the gorilla experiment! Where people counting basketball passes
      completely miss a gorilla walking through. It's wild how selective
      attention works.

Claude: The invisible gorilla is one of the most replicated findings in
       psychology. It shows System 1 isn't just fast - it actively filters
       what you perceive. Did anything in the chapter connect to your
       day-to-day work?

User: Honestly yeah - when I'm debugging, I sometimes miss obvious things
      because I'm so focused on one hypothesis. That's basically the same
      thing.

Claude: That's a great connection - confirmation bias in debugging is
       exactly System 1 taking over. Anything else from this chapter,
       or shall I capture what we discussed?

User: That's about it for tonight.

Claude: Here's what I captured from tonight's session:

       **TLDR additions:**
       - Our thinking runs on two systems: System 1 (fast, intuitive,
         always on) and System 2 (slow, deliberate, lazy) - most mental
         errors come from System 1 operating unchecked
       - Selective attention means we miss obvious things when focused
         elsewhere - the invisible gorilla experiment shows this isn't a
         flaw but how System 1 fundamentally works

       **Chapter 1: The Characters of the Story** (2026-03-05)
       Kahneman introduces the two-system framework that underpins the
       entire book. System 1 operates automatically and quickly with
       little effort or sense of control. System 2 allocates attention
       to effortful mental activities. The invisible gorilla experiment
       demonstrates how System 1's selective attention can cause us to
       miss obvious things - a pattern that shows up in everyday work
       like debugging, where focus on one hypothesis blinds us to
       simpler explanations. Key insight: most people overestimate how
       much they use System 2.

       Look good? (yes / make changes)

User: yes

Claude: Created: Book Reading/Thinking Fast and Slow.md
```

## Example Session - Returning to an Existing Book

```
User: /book-reading

Claude: What book did you read tonight?

User: Thinking Fast and Slow

Claude: [Reads existing file - sees Chapter 1 was covered on 2026-03-05]
       Last time we covered Chapter 1 - "The Characters of the Story".
       Did you read Chapter 2 tonight?

User: Yep, Chapter 2 - Attention and Effort

Claude: What stood out to you most in this chapter?

[... discussion continues, then updates the existing file ...]

Claude: Updated: Book Reading/Thinking Fast and Slow.md
       (added 2 TLDR points, added Chapter 2 notes)
```
