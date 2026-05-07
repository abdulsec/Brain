# Cursor AI Instructions for SaaS Development

## General Prompting Guidelines

1. Be specific about the technology stack and frameworks you want to use
2. Break down complex features into smaller, focused requests
3. Always specify the file you want to work on
4. Request explanations when needed
5. Use version control (Git) commands when managing multiple files

## Example Prompts

### Project Setup
```
Create a new Next.js 14 project with the following:
- TypeScript
- Tailwind CSS
- Authentication system
- PostgreSQL database
- Prisma ORM
Show me the required commands and initial file structure.
```

### Feature Development
```
Help me create a subscription system with:
- Monthly and annual pricing tiers
- Stripe integration
- User dashboard
- Payment history
Please show the necessary components and API routes.
```

### Database Operations
```
Create a Prisma schema for:
- User profiles
- Subscription plans
- Payment history
- Usage analytics
Include the relationships between models.
```

## Best Practices for Different Tasks

### Frontend Development
1. Request component creation with specific props and types
2. Ask for styling using Tailwind classes
3. Request responsive design considerations
4. Ask for proper state management implementation

Example:
```
Create a dashboard component that displays:
- User subscription status
- Usage metrics
- Recent activity
Use Tailwind for styling and make it responsive.
```

### Backend Development
1. Specify API endpoint requirements
2. Request error handling implementation
3. Ask for authentication middleware
4. Request database queries optimization

Example:
```
Create an API endpoint for:
- Fetching user subscription data
- Include error handling
- Add rate limiting
- Implement caching
```

### Database Management
1. Request schema modifications
2. Ask for migration scripts
3. Specify data relationships
4. Request optimization queries

Example:
```
Add a new field to the User model:
- Last login timestamp
- Create the migration
- Update related queries
```

## Troubleshooting Tips

1. When encountering errors:
   ```
   Review the error in [file path] and suggest fixes for:
   [paste error message]
   ```

2. For performance issues:
   ```
   Analyze the performance of [component/function] and suggest optimizations.
   ```

3. For code improvements:
   ```
   Review the code in [file path] for:
   - Best practices
   - Security concerns
   - Performance optimization
   ```

## Project Organization

1. Request file structure organization:
   ```
   Suggest a scalable file structure for:
   - API routes
   - Components
   - Utilities
   - Types
   ```

2. Code splitting:
   ```
   Help me split [component/function] into smaller, reusable parts.
   ```

## Testing

1. Request test creation:
   ```
   Create tests for:
   - API endpoints
   - Components
   - Utilities
   Include both unit and integration tests.
   ```

## Deployment

1. Request deployment configuration:
   ```
   Help set up deployment configuration for:
   - Vercel
   - Environment variables
   - Database connection
   ```

## Important Notes

1. Always commit your changes frequently
2. Review generated code before implementing
3. Keep security best practices in mind
4. Document your code as you build
5. Maintain consistent coding standards

## Tips for Better Results

1. Provide context about your project when asking for help
2. Be specific about the problem you're trying to solve
3. Ask for explanations when you don't understand something
4. Request code comments for complex implementations
5. Break down large features into smaller, manageable tasks

Remember to adjust these prompts based on your specific needs and technology stack. Cursor AI works best when given clear, specific instructions with proper context.
