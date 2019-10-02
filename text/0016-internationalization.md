
# Internationalization

- Start Date: 2019/09/26
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/17
	

## Definitions

Localization refers to the adaptation of a product, application or document content to meet the language, cultural and other requirements of a specific target market (a locale).

It is sometimes written as l10n, where 10 is the number of letters between l and n.

  

Internationalization is the design and development of a product, application or document content that enables easy localization for target audiences that vary in culture, region, or language. It is often written i18n, where 18 is the number of letters between i and n.

  

Transifex is a cloud-based localization platform built to help you manage the translation and localization of your app, website, video subtitles, and more. It acts as a repository for your content (think GitHub for translation) and includes tools for developers to get that content into Transifex automatically. Transifex also provides translators a web interface to submit translations.

  

## Decision, including impact on distributions

OpenMRS is implemented throughout the world and is used by speakers of many different languages. As not every user speaks English as a primary language, we will:

1.  Provide an ESM module to translate content in the UI to the user’s specified language.
    
2.  The default language is English, this should be made visible and explicit in the code.
    
3.  The ESM translation module will be able to translate content **without** connectivity.
    
4.  Presently, OpenMRS uses Transifex ([https://www.transifex.com/openmrs/OpenMRS/dashboard/](https://www.transifex.com/openmrs/OpenMRS/dashboard/)) to manage translations. We will continue to use Transifex to support translations for the MF project.
    
5.  Possibly find ways to reuse OpenMRS module translations found in the `message.properties` file.

7.  Update mechanism: Once translations are marked as complete, create a mechanism to publish the translated strings to a github repo. 
    

## Reason for decision

- Most distributions are deployed to areas with no to intermittent connectivity, thus the translations module should be completely offline.

- English is the lingua franca and spoken in most parts of the world.

- Reasons for choosing Transifex:
	 - Transifex is the most popular internationalization SaaS and offers tools that would make the translations workflow smoother.
	 - The community is already familiar using transifex for translation services.


## Alternatives
- No translation service at all and default to English for all frontend modules.
-  Use another service provider built specifically for frontend applications such as [react-i18next](https://github.com/i18next/react-i18next)
- We could store the translations with each `esm` module instead of having a translations `esm` module.
- Repo to handle translations service: we could have the translations handled by `openmrs-esm-api`


## Common practices (not enforced)


### Definitions

Messages - OpenMRS historically used this word to refer to the content that would be replaced.

  
### Structure of Message File

Here’s an example of how message.properties file looks like:

[https://github.com/openmrs/openmrs-module-legacyui/blob/master/api/src/main/resources/messages.properties](https://github.com/openmrs/openmrs-module-legacyui/blob/master/api/src/main/resources/messages.properties)


### Existing API

Here is a [link](https://github.com/openmrs/openmrs-core/blob/master/api/src/main/java/org/openmrs/messagesource/MessageSourceService.java) to the github repo for the interface to the java MessageSourceService in OpenMRS core. It has four methods:

    public String getMessage(String s);
    
    public MutableMessageSource getActiveMessageSource();
    
    public  void  setActiveMessageSource(MutableMessageSource activeMessageSource);
    
    public  void  setMessageSources(Set<MutableMessageSource>  availableMessageSources);

  

### Proposed API

1.  `setPreferredLocale()`: sets the preferred locale of the user, defaults to English if the user has no preferred locale.
    
2.  `getMessages(translation)` : takes argument `es-mx` and returns all the messages for that locale.
    
3.  `translate(messageVariableName, englishMessage, target-language)` : takes a message and translates it to the target language, if target language is not found, it returns the englishMessage
    

  
  

### Example Code

    import { getCurrentUser } from "@openmrs/esm-api";
    
      
    
    const language = getCurrentUser().getLanguage(); //pseudocode
    
    function getMessages(language) {
    
    import messages from “@openmrs/esm-intl-messages”
    
    }
    
      
    
    const title = translate(“Patient Search”, es-mx);

