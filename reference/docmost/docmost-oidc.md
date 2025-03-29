You’re absolutely right—making the SSO integration generic for any OpenID Connect (OIDC) provider is a better approach than tying it specifically to Keycloak. This aligns with flexibility and reusability, allowing users to plug in Keycloak, Auth0, Okta, or any other OIDC-compliant provider. I’ll revise the variable names and configuration to be provider-agnostic, keeping them consistent with Docmost’s existing conventions (uppercase with underscores) while ensuring they’re generic enough for OIDC.

Here’s the updated solution:

---

### Step 1: Modify the Codebase for Generic OIDC

#### 1. Update Gemfile
No changes needed—the gems already support generic OIDC:
```ruby
# Gemfile
gem 'devise' # Already present
gem 'omniauth-openid-connect'
gem 'omniauth-rails_csrf_protection'
```

#### 2. Configure Devise with Generic OIDC
Update `config/initializers/devise.rb` to use generic environment variables and a dynamic provider name:
```ruby
# config/initializers/devise.rb
Devise.setup do |config|
  # Existing config...

  config.omniauth :openid_connect,
    {
      name: :oidc, # Generic name for the provider
      scope: [:openid, :profile, :email],
      issuer: ENV['OIDC_ISSUER'], # e.g., http://keycloak:8080/auth/realms/docmost-realm
      client_options: {
        identifier: ENV['OIDC_CLIENT_ID'],
        secret: ENV['OIDC_CLIENT_SECRET'],
        redirect_uri: "#{ENV['APP_HOST']}/users/auth/oidc/callback" # Dynamic callback
      },
      discovery: true # Auto-discover OIDC endpoints if supported
    }
end
```

- `OIDC_ISSUER`: The issuer URL of the OIDC provider (specific to the provider’s realm or tenant).
- `OIDC_CLIENT_ID`: The client ID from the OIDC provider.
- `OIDC_CLIENT_SECRET`: The client secret from the OIDC provider.
- `APP_HOST`: The base URL of the Docmost app (e.g., `http://localhost:3000`).
- The provider name is set to `:oidc` instead of `:keycloak`, keeping it generic.

#### 3. User Model
No changes needed—it’s already provider-agnostic:
```ruby
# app/models/user.rb
class User < ApplicationRecord
  devise :database_authenticatable, :registerable, :recoverable, :rememberable, :validatable
  devise :omniauthable, omniauth_providers: [:oidc]

  def self.from_omniauth(auth)
    where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
      user.email = auth.info.email
      user.password = Devise.friendly_token[0, 20]
      user.name = auth.info.name
    end
  end
end
```

#### 4. OmniAuth Callback Controller
Update the method name to match the generic `:oidc` provider:
```ruby
# app/controllers/users/omniauth_callbacks_controller.rb
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def oidc
    @user = User.from_omniauth(request.env['omniauth.auth'])
    if @user.persisted?
      sign_in_and_redirect @user, event: :authentication
      set_flash_message(:notice, :success, kind: 'OIDC') if is_navigational_format?
    else
      redirect_to new_user_registration_url, alert: 'Authentication failed.'
    end
  end
end
```

- Changed `keycloak` to `oidc` to match the provider name.

#### 5. Routes
No changes needed—it’s already set up for the callbacks controller:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  devise_for :users, controllers: { omniauth_callbacks: 'users/omniauth_callbacks' }
  # Existing routes...
end
```

#### 6. Login View
Update the link text to be generic:
```erb
<!-- app/views/devise/sessions/new.html.erb -->
<h2>Log in</h2>
<%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
  <!-- Existing fields -->
<% end %>

<p><%= link_to 'Log in with SSO', user_oidc_omniauth_authorize_path %></p>
```

- Changed “Log in with Keycloak” to “Log in with SSO” for provider neutrality.

---

### Step 2: Update Docker Configuration

#### 1. Dockerfile
No changes needed—it’s already generic and will install the required gems:
```dockerfile
# Dockerfile
FROM ruby:3.2.2

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client

WORKDIR /app
COPY Gemfile Gemfile.lock ./
RUN bundle install

COPY . .

RUN bundle exec rake assets:precompile

EXPOSE 3000
CMD ["rails", "server", "-b", "0.0.0.0"]
```

#### 2. Update docker-compose.yml
Update the environment variables to be OIDC-generic. Since we’re not tying it to Keycloak, I’ll include Keycloak as an optional service for testing, but the app won’t depend on it:
```yaml
# docker-compose.yml
version: '3'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://postgres:password@db:5432/docmost_development
      - RAILS_ENV=development
      - APP_HOST=http://localhost:3000
      - OIDC_ISSUER=http://keycloak:8080/auth/realms/docmost-realm
      - OIDC_CLIENT_ID=docmost
      - OIDC_CLIENT_SECRET=your-client-secret # Replace with actual secret
    depends_on:
      - db
    volumes:
      - .:/app

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=docmost_development
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Optional Keycloak service for testing
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    ports:
      - "8080:8080"
    environment:
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
    command: start-dev
    # No depends_on link to app, making it optional

volumes:
  postgres_data:
```

- Removed `depends_on: keycloak` from the `app` service to avoid hard dependency.
- Variables renamed to `OIDC_*` for generic OIDC support.
- You can override these variables for any OIDC provider (e.g., Auth0, Okta) by updating the values.

#### 3. External Configuration (Recommended)
Use a `.env` file for flexibility:
```env
# .env
APP_HOST=http://localhost:3000
OIDC_ISSUER=http://keycloak:8080/auth/realms/docmost-realm
OIDC_CLIENT_ID=docmost
OIDC_CLIENT_SECRET=your-client-secret
```

Update `docker-compose.yml`:
```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    environment:
      - DATABASE_URL=postgres://postgres:password@db:5432/docmost_development
      - RAILS_ENV=development
    depends_on:
      - db
    volumes:
      - .:/app
  # db and keycloak services remain the same
```

---

### Step 3: Configure an OIDC Provider (e.g., Keycloak)
For testing with Keycloak:
1. Start the services: `docker-compose up`.
2. Access Keycloak at `http://localhost:8080`, log in with `admin/admin`.
3. Create a realm (e.g., `docmost-realm`).
4. Create a client:
   - Client ID: `docmost` (matches `OIDC_CLIENT_ID`).
   - Access Type: `confidential`.
   - Valid Redirect URIs: `${APP_HOST}/users/auth/oidc/callback` (e.g., `http://localhost:3000/users/auth/oidc/callback`).
   - Copy the `Client Secret` and update your `.env` or `docker-compose.yml`.

For other providers (e.g., Auth0):
- Set `OIDC_ISSUER` to the provider’s issuer URL (e.g., `https://<your-domain>.auth0.com/`).
- Update `OIDC_CLIENT_ID` and `OIDC_CLIENT_SECRET` with the provider’s values.
- Ensure the redirect URI matches in the provider’s settings.

---

### Step 4: Build and Test
1. **Build**:
   ```bash
   docker-compose build
   ```
2. **Run**:
   ```bash
   docker-compose up
   ```
3. Test SSO at `http://localhost:3000` by clicking “Log in with SSO”.

---

### Benefits of This Approach
- **Generic**: Works with any OIDC provider by adjusting `OIDC_ISSUER`, `OIDC_CLIENT_ID`, and `OIDC_CLIENT_SECRET`.
- **Consistent**: Uses Docmost’s uppercase-with-underscores naming (e.g., `APP_HOST`, `DATABASE_URL`).
- **Flexible**: No hardcoded values—everything is configurable via environment variables.
- **Optional Keycloak**: Included for testing but not required, avoiding provider lock-in.

This setup should meet your needs for a customizable, provider-agnostic SSO solution in Docmost. Let me know if you want to tweak anything further!