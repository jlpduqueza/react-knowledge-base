# Forms in React

## Controlled Components

React state is the "single source of truth" for an input's value; every keystroke updates state, and the input's `value` is driven from that state.

```jsx
function NameForm() {
  const [name, setName] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitted:', name);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={e => setName(e.target.value)} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Handling Multiple Inputs

```jsx
function ProfileForm() {
  const [form, setForm] = useState({ email: '', bio: '' });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
  };

  return (
    <form>
      <input name="email" value={form.email} onChange={handleChange} />
      <textarea name="bio" value={form.bio} onChange={handleChange} />
    </form>
  );
}
```

## Checkboxes, Radios, and Selects

```jsx
<input type="checkbox" checked={agreed} onChange={e => setAgreed(e.target.checked)} />

<select value={country} onChange={e => setCountry(e.target.value)}>
  <option value="us">United States</option>
  <option value="ca">Canada</option>
</select>
```

## Controlled vs. Uncontrolled Components

- **Controlled** — React state drives the value (shown above). More code, but full control (validation on every keystroke, conditional disabling, formatting).
- **Uncontrolled** — the DOM itself holds the value; read it via a ref when needed (e.g. on submit), closer to plain HTML forms.

```jsx
function UncontrolledForm() {
  const inputRef = useRef();

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(inputRef.current.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} defaultValue="" />
    </form>
  );
}
```

Uncontrolled inputs are simpler for basic cases (a single submit-time read) but harder to validate or react to as the user types.

## Basic Validation

```jsx
function SignupForm() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!email.includes('@')) {
      setError('Please enter a valid email');
      return;
    }
    setError('');
    // submit...
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={e => setEmail(e.target.value)} />
      {error && <p className="error">{error}</p>}
      <button type="submit">Sign up</button>
    </form>
  );
}
```

## Form Libraries (Worth Knowing)

Hand-rolled validation gets unwieldy for larger forms. Common libraries:

- **React Hook Form** — uncontrolled-by-default, minimal re-renders, popular for performance.
- **Formik** — controlled, mature, integrates well with Yup for schema-based validation.
- **Zod / Yup** — schema validation libraries commonly paired with the above.

```jsx
// React Hook Form example
function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email', { required: true })} />
      {errors.email && <span>Email is required</span>}
      <button type="submit">Log in</button>
    </form>
  );
}
```

## Key Takeaways

- Always call `e.preventDefault()` in `onSubmit` to stop the browser's native full-page reload.
- Prefer controlled inputs for anything needing live validation/formatting; uncontrolled is fine for simple, submit-only forms.
- Reach for a form library once a form has more than a handful of fields or needs schema validation.
