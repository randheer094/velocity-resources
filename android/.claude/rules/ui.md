# Resources & accessibility

- User-visible text lives in `res/values/strings.xml`; counts use
  `<plurals>`; format args go through `<xliff:g>`.
- Dimensions in `dp` (layout) / `sp` (text); never `px`.
- Every interactive element has a `contentDescription` or
  `Modifier.semantics`; touch targets ≥ 48dp.
- Decorative imagery sets `contentDescription = null`.
- Strings, dimens, and colors live in `res/`; no hardcoded user-
  visible strings or magic dp/sp values in code.
- Image loading goes through Coil (`AsyncImage` /
  `rememberAsyncImagePainter`) with an explicit placeholder and
  error painter — never `Image(painter = rememberDrawablePainter)`.
