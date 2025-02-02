# MobX Light Form

### ✨ MobX Form State Management with automatic validation

Seperate subjects which manage form data to prevent duplication of data and ensure consistency.<br>
The basic principle of responsibility allocation is to allocate responsibility to objects with information that can perform responsibility, so that the form can take charge of the transformation of data to be sent from the client to the server.

### [Demo](https://codesandbox.io/s/mobx-light-form-demo-wmnbd?file=/src/views/PersonBody.tsx)

<div align="center">
  <img src="https://user-images.githubusercontent.com/23455736/153737842-44843682-c1df-4d50-b3ec-54704598a01f.png" alt="예시 이미지" width="680">
</div>

### 🔍️ Features

- Form update and reset
- Check form is valid or has some errors
- To be added

### ⚙ Install

#### npm

```
npm install mobx-light-form
```

#### yarn

```
yarn add mobx-light-form
```

### 🚀 Quick Start

#### 1-a. Define Form

```typescript
import { makeObservable, observable } from 'mobx';

import Form, { FieldSource } from 'mobx-light-form';

export default class PersonForm extends Form {
  public name: FieldSource<string>;

  constructor() {
    super();

    this.name = this.generateField<string>({
      key: 'name', // Required: Should be same as member field
      label: 'Name',
      isRequired: true, // Should not be empty
      value: '', // Default value,
      validation: [
        /^Pewww.*$/, // Can be Regex or
        (v: string) => [ // Custom function - () => [boolean, ErrorMessage | undefined]
          v === 'Pewwwww',
          "Should be 'Pewwwww'"
        ]
      ]
    });

    makeObservable(this, {
      name: observable
    });
  }

  public toDto() { // Convert data to be sent to the server here.
    return {
      personName: this.name.value
    };
  }
}
```

#### 1-b. Define Form with array value

> Define another form to be an item of array.

```typescript
import { makeObservable, observable } from 'mobx';

import Form, { FieldSource } from 'mobx-light-form';

export default class BookForm extends Form {
  public name: FieldSource<string>;

  public author: FieldSource<string>;

  constructor() {
    super();

    this.name = this.generateField<string>({
      key: 'name',
      label: 'Book Name',
      value: '',
      isRequired: true
    });

    this.author = this.generateField<string>({
      key: 'author',
      label: 'Author',
      value: ''
    });

    makeObservable(this, {
      name: observable,
      author: observable
    });
  }
}
```

```typescript
import { makeObservable, observable, action } from 'mobx';

import Form, { FieldSource } from 'mobx-light-form';

import BookForm from 'src';

export default class PersonForm extends Form {
  // ...other fields
  public favoriteBooks: BookForm[];

  constructor() {
    super();

    this.favoriteBooks = [];

    makeObservable(this, {
      favoriteBooks: observable,
      addBook: action,
      clearBooks: action
    });
  }

  public addBook() { // If you need, override or create form methods.
    this.favoriteBooks.push(new BookForm());
  }

  public clearBooks() {
    this.favoriteBooks = [];
  }
}
```

#### 2. Register form in store

```typescript
import PersonForm from 'src';

export default class PersonStore {
  public personForm: PersonForm;

  constructor() {
    this.personForm = new PersonForm();
  }
}
```

#### 3. Handle Input values

```tsx
// Please see Demo

import React, { useCallback } from 'react';
import { observer } from 'mobx-react';

import { usePersonStores } from '../stores/PersonProvider';

import { Input, Button } from '../components';

const PersonBody = observer(() => {
  const { personStore } = usePersonStores(); // The way you get mobx store
  const { personForm: form } = personStore;

  const handleChange = useCallback(
    (fieldName: string) => (value: string) => {
      form.update({
        [fieldName]: value
      });
    },
    [form]
  );

  const handleReset = useCallback(() => {
    form.reset();
  }, [form]);

  const handleBookAdd = useCallback(() => {
    form.addBook();
  }, [form]);

  const handleBooksClear = useCallback(() => {
    form.clearBooks();
  }, [form]);

  const handleSubmit = useCallback(() => {
    console.log('Submit Result: ', form.toDto());
  }, [form]);

  return (
    <Wrapper>
      <Input
        label={form.name.label}
        value={form.name.value}
        placeholder="Write name"
        onChange={handleChange("name")}
      />
      {!!form.errors.name && (
        <ErrorMessage>{form.errors.name}</ErrorMessage>
      )}
      {form.favoriteBooks.map((f) => (
        <FavoriteBookWrapper key={f.__id}>
          <Input
            placeholder="Write author"
            label={f.author.label}
            value={f.author.value}
            onChange={(value) => {
              f.update({
                author: value
              });
            }}
          />
          <Input
            placeholder="Write book name"
            label={f.name.label}
            value={f.name.value}
            className="book-name"
            onChange={(value) => {
              f.update({
                name: value
              });
            }}
          />
        </FavoriteBookWrapper>
      ))}
      <StyledButton onClick={handleReset}>Reset</StyledButton>
      <StyledButton onClick={handleBookAdd}>Add Book</StyledButton>
      <StyledButton onClick={handleBooksClear}>Clear Books</StyledButton>
      <StyledButton onClick={handleSubmit} disabled={!form.isValid}>
        Submit
      </StyledButton>
    </Wrapper>
  );
});

export default PersonBody;
```
