### Application context

Before I provide my solution to L7, I will explain the premise of the application.

The application Pairdraw is a doodle sharing application. In the application the user is supposed to form pairs with friends by sharing invite codes; once a `pair` is formed the users can share between themselves doodles.

In this milestone submission I will be demonstrating how to Dockerize a node js application and automate deployment using CI/CD pipelines.

To showcase this we will be using the backend sub repo of the Pairdraw application which is built in node.js 

### Internationalization and Localization in React

Outlined are the steps taken to implement internationalization and localization features in a React. The goals include translating dynamic content, localizing date and time formats, and enabling users to switch between different locales seamlessly.

### Collaborative Translation Tool

Firstly we need a collaborative translation tool, For this we can use lots of tools, some of these are POEditor, etc. Once set up collaborators can add different languages on their own via a common tool interface.

As an addition to these we can use a headless CMS as a store for different languages, something like sanity would be apt for the task 
### Translation for every language

Translations for languages are stored in JSON files within the project structure. JSON files contain key-value pairs where the keys represent the original text and the values represent the translated text. For example, for English and German translations:

```json
// de.json
{
	{
		"home": "heim",
		"new pair": "neues paar",
		"pairlist": "paarliste",
		"join pair": "paar verbinden",
		"free draw": "kostenlose auslosung"
	}

	{
		"create new pair": "neues paar erstellen",
		"view existing pairs": "vorhandene paare anzeigen"
	}
}
```

### Setting up i18n in react 

to install the necessary dependencies

```bash
npm install i18next react-i18next i18next-browser-languagedetector
```

To initialize i18next we do the following

```ts
import i18n from "i18next";
import LanguageDetector from "i18next-browser-languagedetector";
import { initReactI18next } from "react-i18next";

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    debug: true,
    resources: {
      en: { ...enJson },
      de: { ...deJson },
      // add more languages

    },
    fallbackLng: "en",
  });
```

### Dynamic Translation

Within React , we utilize the `useTranslation` hook in react-i18next to access translations we have defined in our JSON files. For example consider the following 

```tsx
import { useTranslation } from "react-i18next";

const { t } = useTranslation();


const NewPair: React.FC<NewPairProps> = () => {
	// hooks and other logic
	return (
    <>
      <Navbar />
      <div className="max-w-md mx-auto mt-10 p-6 bg-gray-100 rounded-lg shadow-xl">
        <h2 className="text-xl font-semibold mb-4">{t("Create New Pair")}</h2>
        <form onSubmit={handleSubmit}>
          <div className="mb-4">
            <label htmlFor="pairID" className="block text-gray-700">
              {t("Enter Pair ID:")}
            </label>
            <input
              type="text"
              id="pairID"
              value={pairID}
              onChange={(e) => setPairID(e.target.value)}
              className="mt-1 p-2 w-full border rounded-md focus:outline-none focus:border-blue-500"
            />
          </div>
          <button
            type="submit"
            className="bg-blue-500 text-white py-2 px-4 rounded-md hover:bg-blue-600 focus:outline-none focus:bg-blue-600"
          >
            {t("Submit")}
          </button>
        </form>
      </div>
    </>
  );
}
```

### Localization

To localizing date and time formats, we use the `Intl.DateTimeFormat` API provided by JavaScript. This ensures that date and time formats are consistent with the convention followed by the users geographical custom:

```tsx
const dFormatter = new Intl.DateTimeFormat(i18n.language, {
  year: "numeric",
  month: "long",
  day: "numeric",
});

const tFormatter = new Intl.DateTimeFormat(i18n.language, {
  hour: "numeric",
  minute: "numeric",
  second: "numeric",
});

return (
  <div>
    Date: {dFormatter.format(new Date())}
    <br />
    Time: {tFormatter.format(new Date())}
  </div>
);
```

### Switch Locales

To enable users to switch between locales, we either provide a UI component, typically a dropdown menu or automatic location detection via browser location api and permissions, where users can select their preferred language. 

```tsx
// example for dropdown
const handleLanguageChange = (e) => {
  const selectedCode = e.target.value;
  const lang = availableLanguages.find((lang) => lang.code === selectedCode);
  setSelectedLang(lang || availableLanguages[0]);
};

<select value={selectedLang?.code} onChange={handleLanguageChange}>
  {availableLanguages.map((lang) => (
    <option key={lang.code} value={lang.code}>
      {lang.name}
    </option>
  ))}
</select>
```

By following the above steps we can successfully internationalize and localize react/next.js applications all while maintaining similar codebases 

### Example

English content
![[Pasted image 20240821185825.png]]

German Content

![[Pasted image 20240821190510.png]]
