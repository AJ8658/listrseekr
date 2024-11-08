CREATE TABLE public.roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now())
);

CREATE TABLE public.users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  first_name TEXT,
  last_name TEXT,
  profile_picture TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now()),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now()),
  is_active BOOLEAN DEFAULT TRUE
);

CREATE TABLE public.user_roles (
  user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
  role_id UUID REFERENCES public.roles(id) ON DELETE CASCADE,
  assigned_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now()),
  PRIMARY KEY (user_id, role_id)
);

CREATE TABLE public.properties (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  listr_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  description TEXT,
  price NUMERIC NOT NULL,
  address TEXT,
  city TEXT,
  state TEXT,
  zip_code TEXT,
  country TEXT,
  latitude NUMERIC,
  longitude NUMERIC,
  property_type TEXT,
  bedrooms INTEGER,
  bathrooms INTEGER,
  square_feet INTEGER,
  amenities TEXT[],
  status TEXT DEFAULT 'Available',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now()),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now())
);

CREATE TABLE public.property_images (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  property_id UUID REFERENCES public.properties(id) ON DELETE CASCADE,
  image_url TEXT NOT NULL,
  uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now())
);

CREATE TABLE public.messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
  receiver_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
  property_id UUID REFERENCES public.properties(id) ON DELETE SET NULL,
  content TEXT NOT NULL,
  sent_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now()),
  is_read BOOLEAN DEFAULT FALSE
);

CREATE TABLE public.favorites (
  user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
  property_id UUID REFERENCES public.properties(id) ON DELETE CASCADE,
  favorited_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now()),
  PRIMARY KEY (user_id, property_id)
);

CREATE TABLE public.reviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reviewer_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
  listr_id UUID REFERENCES public.users(id) ON DELETE SET NULL,
  property_id UUID REFERENCES public.properties(id) ON DELETE SET NULL,
  rating INTEGER CHECK (rating >= 1 AND rating <= 5),
  comment TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now())
);

CREATE TABLE public.saved_searches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
  search_name TEXT,
  search_criteria JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now())
);

CREATE TABLE public.notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
  type TEXT NOT NULL,
  data JSONB,
  is_read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc', now())
);

-- Add necessary indexes
CREATE INDEX idx_properties_listr_id ON public.properties(listr_id);
CREATE INDEX idx_properties_status ON public.properties(status);
CREATE INDEX idx_messages_sender_id ON public.messages(sender_id);
CREATE INDEX idx_messages_receiver_id ON public.messages(receiver_id);
CREATE INDEX idx_property_images_property_id ON public.property_images(property_id);
CREATE INDEX idx_notifications_user_id ON public.notifications(user_id);

-- Add constraints
ALTER TABLE public.properties
    ADD CONSTRAINT price_positive CHECK (price >= 0),
    ADD CONSTRAINT bedrooms_positive CHECK (bedrooms >= 0),
    ADD CONSTRAINT bathrooms_positive CHECK (bathrooms >= 0),
    ADD CONSTRAINT square_feet_positive CHECK (square_feet >= 0);

-- Create updated_at trigger function
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = timezone('utc', now());
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Add triggers for tables with updated_at
CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON public.users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_properties_updated_at
    BEFORE UPDATE ON public.properties
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

-- Enable Row Level Security on tables
ALTER TABLE public.users ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.properties ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.favorites ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.reviews ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.saved_searches ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.notifications ENABLE ROW LEVEL SECURITY;

-- Users policies
CREATE POLICY users_self_view ON public.users
    FOR SELECT TO authenticated
    USING (id = auth.uid() OR EXISTS (
        SELECT 1 FROM public.properties WHERE listr_id = users.id
    ));

CREATE POLICY users_self_update ON public.users
    FOR UPDATE TO authenticated
    USING (id = auth.uid());

-- Properties policies
CREATE POLICY properties_view ON public.properties
    FOR SELECT TO authenticated
    USING (status = 'Available' OR listr_id = auth.uid());

CREATE POLICY properties_create ON public.properties
    FOR INSERT TO authenticated
    WITH CHECK (auth.uid() IS NOT NULL);

CREATE POLICY properties_update ON public.properties
    FOR UPDATE TO authenticated
    USING (listr_id = auth.uid());

CREATE POLICY properties_delete ON public.properties
    FOR DELETE TO authenticated
    USING (listr_id = auth.uid());

-- Messages policies
CREATE POLICY messages_view ON public.messages
    FOR SELECT TO authenticated
    USING (sender_id = auth.uid() OR receiver_id = auth.uid());

CREATE POLICY messages_create ON public.messages
    FOR INSERT TO authenticated
    WITH CHECK (sender_id = auth.uid());

-- Favorites policies
CREATE POLICY favorites_all ON public.favorites
    FOR ALL TO authenticated
    USING (user_id = auth.uid());

-- Reviews policies
CREATE POLICY reviews_view ON public.reviews
    FOR SELECT TO authenticated
    USING (true);

CREATE POLICY reviews_create ON public.reviews
    FOR INSERT TO authenticated
    WITH CHECK (reviewer_id = auth.uid());

CREATE POLICY reviews_update ON public.reviews
    FOR UPDATE TO authenticated
    USING (reviewer_id = auth.uid());

-- Saved searches policies
CREATE POLICY saved_searches_all ON public.saved_searches
    FOR ALL TO authenticated
    USING (user_id = auth.uid());

-- Notifications policies
CREATE POLICY notifications_all ON public.notifications
    FOR ALL TO authenticated
    USING (user_id = auth.uid());

-- Grant necessary permissions to authenticated users
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO authenticated;
GRANT USAGE ON SCHEMA public TO authenticated;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO authenticated;



