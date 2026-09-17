## Create the TypeScript Component

**customDashComponent.ts**
```
import { Component, inject, OnInit } from '@angular/core';
import {
    Query,
    QueryApi,
    QueryResult
} from '@hylandsoftware/hxcs-js-client';
import { QUERY_API_TOKEN } from '@alfresco/adf-hx-content-services/api';

/*
 * Represents a document stored inside a claim folder.
 */
interface ClaimDocument {
    id: string;
    name: string;
    path: string;
    primaryType: string;
    created: string;
    modified: string;
}

/*
 * Represents a claim folder and the documents
 * stored inside that folder.
 */
interface Claim {
    id: string;
    name: string;
    path: string;
    created: string;
    modified: string;
    documents: ClaimDocument[];
}

@Component({
    selector: 'hxp-dashboard',
    templateUrl: './customDashComponent.html',
    styleUrls: ['./customDashComponent.scss']
})
export class customDashComponent implements OnInit {

    /*
     * Angular injects the QueryApi already configured
     * by the HxP application.
     */
    private queryApi = inject<QueryApi>(QUERY_API_TOKEN);

    dashboardTitle = 'Insurance Claims Dashboard';

    claims: Claim[] = [];

    isLoading = true;
    loadError = '';

    ngOnInit(): void {
        this.loadClaims();
    }


    /**
     * Retrieves all claim folders stored directly
     * inside /uidev_claims.
     */
    async loadClaims(): Promise<void> {

        this.isLoading = true;
        this.loadError = '';

        try {

            const claimQuery: Query = {
                query: `
                    SELECT *
                    FROM SysFolder
                    WHERE sys_parentPath = '/uidev_claims'
                `,
                limit: 1000,
                offset: 0
            };

            console.log('Loading claim folders...');

            const response =
                await this.queryApi.getDocumentsByQuery(claimQuery);

            const result: QueryResult = response.data;

            const claimFolders = result.documents ?? [];

            console.log(
                `Found ${claimFolders.length} claim folders.`,
                claimFolders
            );


            /*
             * Retrieve the documents for every claim.
             *
             * Promise.all allows the document queries to execute
             * without waiting for each previous claim to finish.
             */
            this.claims = await Promise.all(

                claimFolders.map(async (folder: any) => {

                    const documents =
                        await this.loadClaimDocuments(folder.sys_path);

                    return {
                        id: folder.sys_id,
                        name: folder.sys_name,
                        path: folder.sys_path,
                        created: folder.sys_created,
                        modified: folder.sys_modified,
                        documents: documents
                    };

                })

            );

            /*
             * Sort claims alphabetically by claim name.
             */
            this.claims.sort((a, b) =>
                a.name.localeCompare(b.name)
            );

            console.log('Finished claims:', this.claims);

        } catch (error) {

            console.error(
                'Unable to load claims:',
                error
            );

            this.loadError =
                'Unable to retrieve claims from the Content Repository.';

        } finally {

            this.isLoading = false;

        }
    }


    /**
     * Retrieves the files stored directly inside
     * a particular claim folder.
     */
    private async loadClaimDocuments(
        claimPath: string
    ): Promise<ClaimDocument[]> {

        const documentQuery: Query = {
            query: `
                SELECT *
                FROM SysContent
                WHERE sys_parentPath = '${claimPath}'
            `,
            limit: 1000,
            offset: 0
        };

        console.log(
            `Loading documents from ${claimPath}...`
        );

        const response =
            await this.queryApi.getDocumentsByQuery(documentQuery);

        const result: QueryResult = response.data;

        const documents = result.documents ?? [];

        console.log(
            `Found ${documents.length} documents in ${claimPath}.`,
            documents
        );

        return documents.map((document: any) => {

            return {
                id: document.sys_id,
                name: document.sys_name,
                path: document.sys_path,
                primaryType: document.sys_primaryType,
                created: document.sys_created,
                modified: document.sys_modified
            };

        });
    }
}
```

### Step 2
**customDashComponent.html**
```
<div class="dashboard">

    <!-- Dashboard Header -->
    <div class="dashboard-header">

        <div>
            <div class="eyebrow">CLAIMS MANAGEMENT</div>

            <h1>{{ dashboardTitle }}</h1>

            <p class="subtitle">
                Claims and supporting documents from the
                HxP Content Repository.
            </p>
        </div>

        <div class="total-claims-card">

            <div class="total-label">
                TOTAL CLAIMS
            </div>

            <div class="total-number">
                {{ claims.length }}
            </div>

            <div class="total-description">
                Claims in repository
            </div>

        </div>

    </div>


    <!-- Loading -->
    @if (isLoading) {

        <div class="message-card">

            <div class="loading-spinner"></div>

            <h2>Loading Claims</h2>

            <p>
                Retrieving claim information from the
                Content Repository...
            </p>

        </div>

    }


    <!-- Error -->
    @else if (loadError) {

        <div class="message-card error-card">

            <h2>Unable to Load Claims</h2>

            <p>
                {{ loadError }}
            </p>

        </div>

    }


    <!-- Claims -->
    @else {

        <div class="section-header">

            <div>
                <h2>Claims</h2>

                <p>
                    Select a claim below to review its
                    supporting documents.
                </p>
            </div>

            <div class="claim-count">
                {{ claims.length }} Claims
            </div>

        </div>


        @if (claims.length === 0) {

            <div class="message-card">

                <h2>No Claims Found</h2>

                <p>
                    There are currently no claims in the
                    uidev_claims repository folder.
                </p>

            </div>

        }


        <div class="claims-grid">

            @for (claim of claims; track claim.id) {

                <div class="claim-card">


                    <!-- Claim Header -->
                    <div class="claim-header">

                        <div>

                            <div class="claim-label">
                                CLAIM
                            </div>

                            <h2>
                                {{ claim.name }}
                            </h2>

                        </div>


                        <div class="document-count">

                            <strong>
                                {{ claim.documents.length }}
                            </strong>

                            <span>
                                Documents
                            </span>

                        </div>

                    </div>


                    <!-- Document Section -->
                    <div class="documents-header">

                        <span>
                            Supporting Documents
                        </span>

                    </div>


                    <div class="documents">

                        @if (claim.documents.length === 0) {

                            <div class="no-documents">

                                No documents have been
                                added to this claim.

                            </div>

                        }


                        @for (
                            document of claim.documents;
                            track document.id
                        ) {

                            <div class="document-row">

                                <div class="document-icon">
                                    📄
                                </div>


                                <div class="document-details">

                                    <div class="document-name">
                                        {{ document.name }}
                                    </div>

                                    <div class="document-type">
                                        {{ document.primaryType }}
                                    </div>

                                </div>

                            </div>

                        }

                    </div>


                    <!-- Claim Footer -->
                    <div class="claim-footer">

                        <span>
                            {{ claim.documents.length }}
                            supporting document(s)
                        </span>

                    </div>

                </div>

            }

        </div>

    }

</div>
```

**customDashComponent.scss**
```
:host {
    display: block;
    background: #f5f7fa;
    min-height: 100vh;
}

.dashboard {
    max-width: 1400px;
    margin: 0 auto;
    padding: 40px;
    font-family: Arial, Helvetica, sans-serif;
    color: #1f2937;
}


/* ----------------------------------
   Dashboard Header
---------------------------------- */

.dashboard-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 30px;
    margin-bottom: 45px;
}

.eyebrow {
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 1.6px;
    color: #667085;
    margin-bottom: 8px;
}

.dashboard-header h1 {
    margin: 0;
    font-size: 36px;
    font-weight: 700;
    color: #172b4d;
}

.subtitle {
    margin-top: 10px;
    color: #667085;
    font-size: 16px;
}


/* ----------------------------------
   Total Claims
---------------------------------- */

.total-claims-card {
    background: white;
    min-width: 180px;
    padding: 22px 28px;
    border-radius: 12px;
    border: 1px solid #e3e7ed;
    box-shadow: 0 3px 10px rgba(0, 0, 0, 0.05);
}

.total-label {
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 1.2px;
    color: #667085;
}

.total-number {
    font-size: 42px;
    line-height: 1;
    font-weight: 700;
    color: #172b4d;
    margin: 8px 0;
}

.total-description {
    font-size: 13px;
    color: #667085;
}


/* ----------------------------------
   Claims Section
---------------------------------- */

.section-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    margin-bottom: 20px;
}

.section-header h2 {
    margin: 0;
    font-size: 24px;
    color: #172b4d;
}

.section-header p {
    margin: 6px 0 0;
    color: #667085;
}

.claim-count {
    background: #e9eef6;
    padding: 7px 13px;
    border-radius: 20px;
    font-size: 13px;
    font-weight: 600;
    color: #344054;
}


/* ----------------------------------
   Claim Grid
---------------------------------- */

.claims-grid {
    display: grid;
    grid-template-columns:
        repeat(auto-fit, minmax(360px, 1fr));
    gap: 22px;
}


/* ----------------------------------
   Claim Card
---------------------------------- */

.claim-card {
    background: white;
    border: 1px solid #e1e6ed;
    border-radius: 12px;
    overflow: hidden;

    box-shadow:
        0 2px 7px rgba(0, 0, 0, 0.04);

    transition:
        transform 0.15s ease,
        box-shadow 0.15s ease;
}

.claim-card:hover {
    transform: translateY(-2px);

    box-shadow:
        0 7px 18px rgba(0, 0, 0, 0.08);
}


/* ----------------------------------
   Claim Header
---------------------------------- */

.claim-header {
    padding: 22px;

    display: flex;
    justify-content: space-between;
    align-items: flex-start;

    border-bottom: 1px solid #edf0f4;
}

.claim-label {
    font-size: 10px;
    letter-spacing: 1.4px;
    font-weight: 700;
    color: #667085;
}

.claim-header h2 {
    margin: 5px 0 0;
    font-size: 20px;
    color: #172b4d;
}

.document-count {
    text-align: center;

    background: #f2f5f9;
    border-radius: 8px;

    padding: 8px 12px;
}

.document-count strong {
    display: block;
    font-size: 18px;
    color: #172b4d;
}

.document-count span {
    display: block;
    font-size: 10px;
    color: #667085;
    margin-top: 2px;
}


/* ----------------------------------
   Documents
---------------------------------- */

.documents-header {
    padding: 14px 22px 8px;

    font-size: 11px;
    font-weight: 700;
    letter-spacing: 0.7px;

    color: #667085;
}

.documents {
    padding: 0 14px 14px;
}

.document-row {
    display: flex;
    align-items: center;
    gap: 12px;

    padding: 11px 8px;

    border-radius: 7px;
}

.document-row:hover {
    background: #f6f8fb;
}

.document-icon {
    display: flex;
    align-items: center;
    justify-content: center;

    width: 36px;
    height: 36px;

    background: #eef2f7;
    border-radius: 7px;

    flex-shrink: 0;
}

.document-details {
    min-width: 0;
}

.document-name {
    font-size: 14px;
    font-weight: 600;

    color: #344054;

    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.document-type {
    margin-top: 3px;
    font-size: 11px;
    color: #98a2b3;
}

.no-documents {
    padding: 20px 8px;

    color: #98a2b3;
    font-size: 13px;

    text-align: center;
}


/* ----------------------------------
   Claim Footer
---------------------------------- */

.claim-footer {
    padding: 12px 22px;

    background: #fafbfc;
    border-top: 1px solid #edf0f4;

    color: #667085;
    font-size: 11px;
}


/* ----------------------------------
   Loading / Errors
---------------------------------- */

.message-card {
    background: white;

    border: 1px solid #e1e6ed;
    border-radius: 12px;

    padding: 50px;

    text-align: center;
}

.message-card h2 {
    margin: 10px 0;
    color: #172b4d;
}

.message-card p {
    color: #667085;
}

.error-card {
    border-color: #f1b5b5;
}

.loading-spinner {
    width: 32px;
    height: 32px;

    margin: 0 auto 15px;

    border: 3px solid #e4e7ec;
    border-top-color: #475467;

    border-radius: 50%;

    animation: spin 0.8s linear infinite;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}


/* ----------------------------------
   Responsive
---------------------------------- */

@media (max-width: 700px) {

    .dashboard {
        padding: 20px;
    }

    .dashboard-header {
        flex-direction: column;
        align-items: stretch;
    }

    .total-claims-card {
        width: auto;
    }

    .claims-grid {
        grid-template-columns: 1fr;
    }

    .section-header {
        align-items: flex-start;
        flex-direction: column;
        gap: 12px;
    }
}
```

Dash-Fix:
```
/*
 * Copyright © 2005 - 2021 Alfresco Software, Ltd. All rights reserved.
 *
 * License rights for this program may be obtained from Alfresco Software, Ltd.
 * pursuant to a written agreement and any use of this program without such an
 * agreement is prohibited.
 */

import { AppConfigService } from '@alfresco/adf-core';
import {
    ContentActionRef,
    ExtensionService
} from '@alfresco/adf-extensions';

import {
    Component,
    effect,
    inject,
    signal
} from '@angular/core';

import { toSignal } from '@angular/core/rxjs-interop';
import { MatDividerModule } from '@angular/material/divider';
import { RouterLink } from '@angular/router';

import {
    IdentityUserService
} from '@alfresco/adf-process-services-cloud';

import { of } from 'rxjs';
import { switchMap } from 'rxjs/operators';


interface HxpHeaderConfig {
    headerColor: string;
    headerTextColor: string;

    application: {
        name: string;
        logo: string;
        headerImagePath: string;
    };

    features?: {
        header?: ContentActionRef[];
    };
}


@Component({
    selector: 'hxp-workspace-header',
    templateUrl: './header.component.html',
    imports: [
        MatDividerModule,
        RouterLink
    ],
})
export class HxpWorkspaceHeaderComponent {

    private readonly extensionService =
        inject(ExtensionService);

    private readonly appConfigService =
        inject(AppConfigService);

    private readonly identityUserService =
        inject(IdentityUserService);


    readonly headerTextColor =
        signal<string>('');

    readonly backgroundColor =
        signal<string>('');

    readonly backgroundImage =
        signal<string>('');

    readonly logoPath =
        signal<string>('');


    readonly config =
        toSignal<HxpHeaderConfig | null>(
            this.extensionService.setup$.pipe(
                switchMap(() =>
                    of<HxpHeaderConfig | null>(
                        this.appConfigService.config
                    )
                )
            ),
            {
                initialValue: null
            }
        );


    landingPageURL = '/portal';


    constructor() {

        /*
         * Load header configuration.
         */
        effect(() => {

            const config = this.config();

            if (!config) {
                return;
            }


            if (config.headerTextColor) {
                this.headerTextColor.set(
                    config.headerTextColor
                );
            }


            if (config.headerColor) {
                this.backgroundColor.set(
                    config.headerColor
                );
            }


            if (config.application?.headerImagePath) {
                this.backgroundImage.set(
                    config.application.headerImagePath
                );
            }


            if (config.application?.logo) {
                this.logoPath.set(
                    config.application.logo
                );
            }

        });


        /*
         * Determine the landing page based
         * on the current user's group.
         */
        this.configureLandingPage();

    }


    private configureLandingPage(): void {

        const currentUser =
            this.identityUserService.getCurrentUserInfo();


        if (!currentUser?.username) {

            console.warn(
                'Unable to determine the current user.'
            );

            this.landingPageURL = '/portal';

            return;
        }


        console.log(
            'This is the user:',
            currentUser.firstName,
            currentUser.lastName
        );


        this.identityUserService.search(
            currentUser.username,
            {
                groups: [
                    'Account Administrators'
                ]
            }
        ).subscribe({

            next: (users) => {

                console.log(
                    'Account Administrator search:',
                    users
                );


                if (users.length > 0) {

                    this.landingPageURL =
                        '/dashboard';

                } else {

                    this.landingPageURL =
                        '/portal';

                }

            },


            error: (error) => {

                console.error(
                    'Unable to determine user group:',
                    error
                );

                this.landingPageURL =
                    '/portal';

            }

        });

    }

}
```

