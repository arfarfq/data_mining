# AI Usage Tracking

The following dictionary was created using AI taken from the project description

```python
dictionary_rows = [
    ('member_id', 'Identifier', 'Unique member identifier', 'Exclude from distances'),
    ('age', 'Numeric', 'Age at extraction', 'Review implausible ages; not birth year'),
    ('gender', 'Categorical', 'Reported gender', 'Profiling context; not default distance input'),
    ('industry', 'Categorical', 'Member industry', 'Normalize aliases; preserve Unknown category'),
    ('country', 'Categorical', 'Member/company country', 'Normalize country aliases'),
    ('is_team_account', 'Binary', 'Team/company account indicator', 'Account-level quantities need interpretation'),
    ('n_colleagues_on_account', 'Numeric', 'Colleagues sharing the account', 'Nonnegative; check team consistency'),
    ('plan_tier', 'Ordinal categorical', 'hot_desk < flex < dedicated_desk < full_time < private_office', 'Order is not equal numerical spacing'),
    ('plan_days_per_week', 'Numeric', 'Weekly entitled attendance days', 'Review zero; cannot exceed seven'),
    ('membership_start', 'Date', 'First membership start', 'Mixed ISO and day/month/year formats'),
    ('membership_end', 'Date', 'First membership end', '1970-01-01 treated as a candidate sentinel'),
    ('last_badge_swipe', 'Date', 'Most recent physical entry', 'Check against start and reference date'),
    ('days_since_last_visit', 'Numeric', 'Reported visit recency', 'Audit mixed reference dates before use'),
    ('total_visits', 'Numeric', 'Visits over the whole membership', 'Nonnegative count; exposure dependence'),
    ('avg_weekly_visits', 'Numeric', 'Average weekly visits over membership', 'Continuous average, not an integer count'),
    ('entitled_visits_last_period', 'Numeric', 'Entitled visits during last membership period', 'Denominator can be zero'),
    ('actual_visits_last_period', 'Numeric', 'Actual visits during last membership period', 'Can exceed entitlement; do not cap automatically'),
    ('meeting_room_hours', 'Numeric', 'Recorded meeting-room hours', 'Missing is not zero'),
    ('event_space_hours', 'Numeric', 'Recorded event-space hours', 'Zero-heavy; review extremes'),
    ('guest_passes_issued', 'Numeric', 'Guest passes issued', 'Missing is not zero'),
    ('uses_meeting_rooms', 'Binary', 'Ever used meeting-room add-on', 'Cross-check hours; time scope may differ'),
    ('uses_event_space', 'Binary', 'Ever used event-space add-on', 'Cross-check hours; time scope may differ'),
    ('uses_guest_passes', 'Binary', 'Ever used guest-pass add-on', 'Cross-check passes; time scope may differ'),
    ('uses_print', 'Binary', 'Used print add-on', 'Stored as 0/1 rather than True/False'),
    ('uses_mail_handling', 'Binary', 'Used mail-handling add-on', 'Keep nullable Boolean type'),
    ('uses_phone_booths', 'Binary', 'Used phone-booth add-on', 'Keep nullable Boolean type'),
    ('uses_day_passes', 'Binary', 'Used day-pass add-on', 'Keep nullable Boolean type'),
    ('uses_storage', 'Binary', 'Used storage add-on', 'Keep nullable Boolean type'),
    ('uses_bike_locker', 'Binary', 'Used bike-locker add-on', 'Rare does not mean erroneous'),
    ('n_renewals', 'Numeric', 'Number of renewals', 'Nonnegative integer; first end is not final churn'),
    ('lifetime_value', 'Numeric', 'Total attributed member revenue', 'Mixed currency formatting; zeros are not missing'),
]
```

The following disctionary was created using AI:

```python
range_bounds = {
    'age': (18, 100),
    'n_colleagues_on_account': (0, None),
    'plan_days_per_week': (0, 7),
    'days_since_last_visit': (0, None),
    'total_visits': (0, None),
    'avg_weekly_visits': (0, None),
    'entitled_visits_last_period': (0, None),
    'actual_visits_last_period': (0, None),
    'meeting_room_hours': (0, None),
    'event_space_hours': (0, None),
    'guest_passes_issued': (0, None),
    'uses_print': (0, 1),
    'n_renewals': (0, None),
    'lifetime_value': (None, None), # Check this bound, should it be unbound?
}
```

The following mapping dictionaries were created with AI:

```python
gender_aliases = {'f': 'Female', 'female': 'Female', 'm': 'Male', 'male': 'Male', 'o': 'Other', 'other': 'Other'}
country_aliases = {
    'pt': 'Portugal', 'portugal': 'Portugal', 'portucal': 'Portugal',
    'uk': 'United Kingdom', 'u.k.': 'United Kingdom', 'gb': 'United Kingdom', 'united kingdom': 'United Kingdom',
    'de': 'Germany', 'germany': 'Germany', 'deutschland': 'Germany',
    'es': 'Spain', 'spain': 'Spain', 'espana': 'Spain', 'españa': 'Spain',
    'fr': 'France', 'france': 'France',
    'nl': 'Netherlands', 'netherlands': 'Netherlands', 'the netherlands': 'Netherlands', 'holland': 'Netherlands',
    'ie': 'Ireland', 'ireland': 'Ireland', 'other (europe)': 'Other (Europe)',
}
industry_aliases = {
    'it': 'Information Technology', 'tech': 'Information Technology', 'information technology': 'Information Technology',
    'marketing & pr': 'Marketing & PR', 'marketing/pr': 'Marketing & PR', 'marketing and pr': 'Marketing & PR',
    'design & creative': 'Design & Creative', 'design/creative': 'Design & Creative', 'design and creative': 'Design & Creative',
    'real estate': 'Real Estate', 'real-estate': 'Real Estate', 'realestate': 'Real Estate',
}

plan_order = ['hot_desk', 'flex', 'dedicated_desk', 'full_time', 'private_office']

plan_tier = {
    "flex": "flex", "Flex": "flex", "FLEX": "flex",
    "hot_desk": "hot_desk", "HOT_DESK": "hot_desk", "Hot Desk": "hot_desk", "hotdesk": "hot_desk",
    "full_time": "full_time", "FULL_TIME": "full_time", "Full Time": "full_time", "fulltime": "full_time",
    "dedicated_desk": "dedicated_desk", "DEDICATED_DESK": "dedicated_desk", "Dedicated Desk": "dedicated_desk", "dedicateddesk": "dedicated_desk",
    "private_office": "private_office", "PRIVATE_OFFICE": "private_office", "Private Office": "private_office", "privateoffice": "private_office",
}
```

This loop for parsing the date was generated by AI:

```python
for feature in date_features:
    tokens = df_clean[feature].astype("string").str.strip()

    parsed = pd.Series(
        pd.NaT, index=df_clean.index, dtype="datetime64[ns]"
    )

    iso_mask = tokens.str.fullmatch(r"\d{4}-\d{2}-\d{2}", na=False)
    slash_mask = tokens.str.fullmatch(r"\d{2}/\d{2}/\d{4}", na=False)

    parsed.loc[iso_mask] = pd.to_datetime(
        tokens.loc[iso_mask], format="%Y-%m-%d", errors="coerce"
    )
    parsed.loc[slash_mask] = pd.to_datetime(
        tokens.loc[slash_mask], format="%d/%m/%Y", errors="coerce"
    )

    invalid = tokens.notna() & parsed.isna()

    if invalid.any():
        raise ValueError(
            f"Invalid dates in {feature}: "
            f"{tokens.loc[invalid].unique().tolist()}"
        )

    df_clean[feature] = parsed
```
