**BasicWithID**:

```javascript
/^={20} Question ={20} *((?:(?!={20} Answer ={20})[\s\S])+)\n*={20} Answer ={20} *((?:(?!={20} Id ={20})[\s\S])+)\n*={20} Id ={20} *([\s\S]*?)(?=(\n---\n\nDECK INFO|\n?(?:<!--)?(?:ID: (\d+).*)))/gm
```