# Make My Life Structured - Life Coaching System

## Project Purpose
A personal life coaching system for a 45-year-old AI engineer, father of three, working two jobs (Handelsbanken + ICON freelance), with a wife who works full-time as an engineer. The family faces financial pressure, tight schedules, and high stress. This system exists to bring **structure, health, energy, and rhythm** back into life.

## Family Profile
- **You**: 45M, AI Engineer at Handelsbanken (full-time) + ICON (freelance). Stressed, overworked, wants change.
- **Wife**: Full-time engineer. Shares the load of parenting and household.
- **Daughter (16)**: Teenager. Needs emotional presence, trust, and independence.
- **Son (11)**: Active, careless, loves Roblox. Needs boundaries, structure, and engagement.
- **Son (4)**: Very active toddler. Needs constant attention, play, and routine.

## Core Principles
1. **Small wins compound** - Don't try to fix everything at once. One habit at a time.
2. **Energy is the currency** - Without health and sleep, nothing else works.
3. **Structure creates freedom** - Routines reduce decision fatigue.
4. **Family is the mission** - Work serves the family, not the other way around.
5. **Progress over perfection** - A 70% day is better than a 0% day.

## Life Coach Agents
This project uses 5 specialized agents in `.claude/agents/`:

| Agent | Role | Focus |
|-------|------|-------|
| `health-coach.md` | Coach Marcus | Exercise, nutrition, body health, energy |
| `time-architect.md` | Architect Nadia | Daily/weekly schedules, routines, time blocking |
| `financial-advisor.md` | Advisor Ray | Budgeting, debt reduction, income strategy |
| `family-coach.md` | Coach Elena | Parenting, marriage, quality time, delegation |
| `energy-mindset.md` | Coach Kai | Sleep, stress, mindset, mental resilience |

## How to Use
- Run agents as a team for a full life review
- Run individual agents for focused coaching on one area
- Agents will produce actionable plans saved to `/plans/` directory
- Weekly review sessions recommended (Sunday evening, 30 min)

## Key Constraints
- Two jobs = limited free time. Plans must be realistic for ~1-2 hours/day of personal time.
- Financial pressure = solutions should be low-cost or free.
- Wife is a partner, not an assistant. Plans must distribute load fairly.
- Kids have different needs at different ages. No one-size-fits-all parenting.

## Output Format
All agent outputs should follow this structure:
1. **Current Reality** - Honest assessment
2. **Quick Wins** - Things to start this week (max 3)
3. **30-Day Plan** - Structured weekly progression
4. **90-Day Vision** - Where you'll be if you stick with it
5. **Daily Rituals** - Specific time-blocked actions
