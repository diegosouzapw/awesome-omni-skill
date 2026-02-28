---
name: ai-recommendations
description: AI-powered recommendation engine specialist using Google AI SDK for personalized content suggestions
---

# AI Recommendations Specialist

## Instructions
When working with AI recommendations:

1. **Generating Recommendations**
   - Collect user preferences from database
   - Build context with liked/disliked content
   - Include favorite actors and directors
   - Use structured prompts for consistent output

2. **AI Integration**
   - Configure Google AI SDK in `src/lib/ai/recommendations.ts`
   - Use structured output for reliable parsing
   - Handle API errors and rate limits
   - Implement fallback recommendations

3. **Smart Filtering**
   - Exclude already recommended content
   - Filter out user's disliked items
   - Ensure content is available
   - Apply content rating restrictions

4. **Performance**
   - Cache recommendations to reduce API calls
   - Batch multiple user requests
   - Generate recommendations asynchronously
   - Store results for quick retrieval

## Examples

**Generating recommendations:**
```typescript
import { generateRecommendations } from '@/lib/ai/recommendations'

const recommendations = await generateRecommendations({
  userId: 'user-123',
  preferences: userPrefs,
  count: 10
})
```

**Building AI prompt context:**
```typescript
const context = {
  likedContent: user.preferences.liked,
  dislikedContent: user.preferences.disliked,
  favoritePeople: user.preferences.people,
  recentlyRecommended: user.recommendations,
  genres: user.favoriteGenres
}
```

**Handling AI response:**
```typescript
const response = await model.generateContent(prompt)
const { text } = response.response
const recommendations = JSON.parse(text)
// Validate and filter recommendations
```