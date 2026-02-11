# Contributing to Mercan

Thank you for your interest in contributing to Mercan! This guide will help you get started.

## Ways to Contribute

1. **Add New Honeypots**: Follow the role template pattern
2. **Improve Documentation**: Fix typos, add examples, clarify instructions
3. **Report Bugs**: Open issues with detailed information
4. **Submit Pull Requests**: Fix bugs or add features
5. **Share Feedback**: Let us know how you're using Mercan

## Adding a New Honeypot

The easiest way to contribute is by adding a new honeypot. See `docs/ROLE_TEMPLATE.md` for detailed instructions.

Quick steps:
1. Fork the repository
2. Create role following the template pattern
3. Test deployment locally
4. Submit pull request with:
   - Role files (4 files minimum)
   - Registry entry in `group_vars/all.yml`
   - Documentation updates in README.md

## Development Guidelines

### Code Style

- **YAML**: Use 2-space indentation
- **Jinja2**: Use consistent spacing around variables
- **Comments**: Add descriptive comments for complex logic
- **Naming**: Use lowercase with underscores for consistency

### Testing

Before submitting a PR:

1. **Syntax Check**:
   ```bash
   ansible-playbook playbooks/deploy.yml --syntax-check
   ```

2. **Local Testing**:
   ```bash
   ansible-playbook playbooks/deploy.yml --tags your_honeypot --check
   ```

3. **Full Deployment**:
   Test on a VM or container with a supported OS

### Documentation

- Update README.md if adding features
- Create/update documentation in `docs/`
- Add comments in code for complex operations
- Update port mapping table if adding honeypots

## Pull Request Process

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b feature/new-honeypot`
3. **Commit** your changes with clear messages
4. **Test** thoroughly on at least one supported OS
5. **Push** to your fork
6. **Submit** a pull request with:
   - Clear description of changes
   - Testing performed
   - Screenshots if applicable

### PR Requirements

- [ ] All playbooks pass syntax check
- [ ] Code follows existing patterns
- [ ] Documentation updated
- [ ] Tested on at least one OS
- [ ] No merge conflicts

## Reporting Bugs

When reporting bugs, include:

- **OS and Version**: Ubuntu 22.04, CentOS 8, etc.
- **Ansible Version**: Output of `ansible --version`
- **Steps to Reproduce**: Exact commands run
- **Expected Behavior**: What should happen
- **Actual Behavior**: What actually happened
- **Error Messages**: Full error output
- **Configuration**: Relevant parts of `group_vars/all.yml`

## Feature Requests

We welcome feature requests! When requesting:

- Explain the use case
- Describe expected behavior
- Provide examples if possible
- Consider submitting a PR if you can implement it

## Code of Conduct

- Be respectful and professional
- Welcome newcomers
- Focus on constructive feedback
- Assume good intentions

## Questions?

- Open an issue for questions
- Check existing documentation first
- Be specific about your problem

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for contributing to Mercan! 🍯
