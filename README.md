# clerk-custom-flows-skill

This repo contains a custom [Agent Skill](https://agentskills.io) that helps when building custom authentication flows using Clerk. Currently, it's focused on building using the `useSignIn()` and `useSignUp()` hooks in React applications.

## Installation

### Claude Code

You can register this repository as a Claude Code Plugin marketplace by running the following command in Claude Code:

```
/plugin marketplace add https://github.com/dstaley/clerk-custom-flows-skill
/plugin install custom-flows@clerk-custom-flows-skills
```

After installing the plugin, you can use the skill by just mentioning it. For instance, you can ask Claude Code to do something like: "Use the custom flows skill to build a sign-in page that uses email and password".
