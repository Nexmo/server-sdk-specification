# Release Checklist

Whenever a new release is needed for a server SDK, please run through this checklist to make sure everything we need is ready to go. Feel free to copy/paste this into an issue to track progress for a new release.

## Minor/Patch Release

- [ ] Make sure that all tests and CI checks are green
- [ ] Update version numbers where needed
- [ ] Update any needed code snippets
- [ ] Update README/documentation as needed
- [ ] Follow the release process for the SDK as outlined in the wiki
- [ ] Create and publish a release in Github (https://keepachangelog.com/en/1.0.0/ as an example)
- [ ] Write any needed content
  - [ ] Write up a release blog post outlining new features/changes
  - [ ] Schedule tweets
- [ ] Update JIRA with “Done” status

## Major Release

- Inherit from the Minor/Patch release list
- [ ] Determine a release date
  - [ ] Write up a release blog post outlining new features/changes
  - [ ] Schedule tweets
- [ ] Make sure that all downstream projects are updated to use the new major version

