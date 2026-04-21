## Doc
### useForm
```
useForm(): a react hook
You call it once at the top of your component, and it gives you everything to build a form.

example:
// 1. Import the hook
import { useForm } from "react-hook-form";

function MyForm() {
  // 2. CALL THE HOOK ✅
  const form = useForm();

  return (
    <form>
      <input {...form.register("name")} />
      <button onClick={form.handleSubmit(onSubmit)}>
        Submit
      </button>
    </form>
  );
}
```
### UseFormReturn
```
UseFormReturn = the type of the object you get from useForm()

type UseFormReturn<TFormValues> = {
  // Functions you use
  register: (name: keyof TFormValues) => any;
  handleSubmit: (onSubmit: (data: TFormValues) => void) => any;
  setValue: (name: keyof TFormValues, value: any) => void;
  reset: () => void;
  watch: (name: keyof TFormValues) => any;

  // Form state
  formState: {
    errors: Partial<Record<keyof TFormValues, any>>;
    isDirty: boolean;
    isSubmitting: boolean;
    isValid: boolean;
  };

  // Other tools
  control: any;
  getValues: () => TFormValues;
};

```
### register
```
{...register("input_text")}

register is a function from react-hook-form. 

after this:
form data = {
  input_text: "Hello, this is my content"
}

//use it this way:
const onSubmit = (data) => {
  // 👇 HERE IS YOUR VALUE
  console.log(data.input_text);
};


register("input_text") RETURNS:
{
  name: "input_text",
  onChange: (e) => saveValue(e.target.value), // tracks typing
  onBlur: () => markFieldAsTouched(),          // validation
  ref: (element) => { ... }                    // connects to form
}

```