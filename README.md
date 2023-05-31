# phraseApp-customLoead
this method call api phraseapp first if the call is not successful it call's the local files 

> file name customLoader.ts

``` TS

import { HttpClient } from '@angular/common/http';
import { TranslateLoader } from '@ngx-translate/core';
import { forkJoin, Observable, of } from 'rxjs';
import { catchError, map } from 'rxjs/operators';


export class CustomLoader implements TranslateLoader {
    constructor(
        private http: HttpClient
    ) { }

    public getTranslation(lang: string): Observable<any> {
        const sourceCalendar = this.sourceCalendar();
        const sourceOne = this.sourceOne(lang);
        const sourceTwo = this.sourceTwo(lang);

        return forkJoin([
            sourceOne.pipe(
                catchError(() => {
                    // If sourceOne fails, switch to sourceTwo
                    return sourceTwo;
                })
            ),
            sourceCalendar
        ]).pipe(
            map(([res1, res2]: [any, any]) => {
                return this.formatResponse(res1, res2);
            }),
            catchError(() => {
                // If both sourceOne and sourceTwo fail, return an empty object
                return of({});
            })
        );
    }

    private sourceOne(lang: string) {
        return this.http.get(
            `${JSON.parse(String(localStorage.getItem('config')))
                ?.PHRASE_APP_BASE_URL}locales/${lang}/download?file_format=json`
        );
    }

    private sourceTwo(lang: string): Observable<object> {
        return this.http.get(`./assets/i18n/${lang}.json`);
    }

    private sourceCalendar(): Observable<object> {
        return this.http.get(`./assets/i18n/primeng.translate.json`);
    }

    private formatResponse(res: any, sourceCalendar: any) {
        let tempRes: any = {};
        Object.entries(res).forEach(
            ([key, value]: [key: string, value: any]) => {
                tempRes[key] = (typeof value === 'object') ? value.message : value;
            }
            );
        // Combine the two translation objects
        Object.entries(sourceCalendar).forEach(
            ([key, value]: [key: string, value: any]) => {
                tempRes[key] = value;
            }
        );
        return tempRes;
    }
}

```
> file name app.module.ts

```TS
TranslateModule.forRoot({
      defaultLanguage: 'en',
      loader: {
        provide: TranslateLoader,
        useClass: CustomLoader, <-------------
        deps: [HttpClient],
      }
    }),
```
