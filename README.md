Suspense is used for show `loading` state
For example,
```HTML
<template>
	<Suspense>
		<template #default> // what is template default
			<DisplayComponent />
		</template>

		<template #fallback>
			<icon loading>
		</template>
	</Suspense>

</template>
```

Ref:
- https://www.youtube.com/watch?v=GQpKm_CNH4A (He puts error in \#default slot)
- https://www.youtube.com/watch?v=r9OmM2FpcC8 (He puts error in \#fallback slot, he mentions async component)
- https://gemini.google.com/app/8234a49d327b3b7b
