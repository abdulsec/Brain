What's up @Critical Thinker's! 

Today I'm releasing an exclusive new script that I created to help you identify potential programs to hack on  

It uses your HackerOne session to pull down all your programs and their bounty stats data, and looks at a few key metrics:

avg high payout vs max high ratio ≥ 75%
avg crit payout vs max crit ratio ≥ 75%
number of highs submitted ≥ 40%
number of crits submitted ≥ 40%
max payout > highest max crit bounty
high + crit percentage (combined) ≥ 40%

With this data, you should be able to spot programs that are more anomalous in terms of their payout consistency, when compared to other programs!

To use the tool, first make sure you have installed the python dependencies:

pip install requests tqdm


Then you just run it like any other Python program:

usage: find_programs.py [-h] [-s SESSION] [-t TOKEN] [-f] [-m MIN_REQ] [-c]

options:
  -h, --help            show this help message and exit
  -s SESSION, --session SESSION
                        __Host-session cookie value from hackerone.com
  -t TOKEN, --token TOKEN
                        X-Csrf-Token value, you can fetch via: document.querySelector("meta[name=csrf-token]").getAttribute("content")
  -f, --force           Force re-fetch list of programs and program data
  -m MIN_REQ, --min_req MIN_REQ
                        Minimum number of identified features required to highlight a program in the output (default=1)
  -c, --check_auth      HackerOne auth check (for debug)


On the first run, you will want to provide --session and --token so you can fetch and cache all your program names and data.

Let me know if you have any questions, feedback, or improvements! Happy hacking



```python
import argparse
import requests
from tqdm import tqdm
import json
import os

# __Host-session cookie from a /graphql request
HOST_SESSION_COOKIE = ''
# x-csrf-token value: document.querySelector("meta[name=csrf-token]").getAttribute("content")
CSRF_TOKEN = ''

SESSION = requests.Session()

ME_QUERY = '''query MeQuery {
  me {
    id
  }
}'''

PROGRAM_HANDLES_QUERY = '''query DiscoveryQuery($query: OpportunitiesQuery!, $filter: QueryInput!, $from: Int, $size: Int, $sort: [SortInput!], $post_filters: OpportunitiesFilterInput) {
  opportunities_search(
    query: $query
    filter: $filter
    from: $from
    size: $size
    sort: $sort
    post_filters: $post_filters
  ) {
    nodes {
      ... on OpportunityDocument {
        handle
      }
    }
    total_count
  }
}
'''

TEAMPROFILE_QUERY = '''query TeamProfile($handle: String!) {
  team(handle: $handle) {
    ...BountyTable
    ...ProfileMetrics
  }
}

fragment BountyTable on Team {
  profile_metrics_snapshot {
    average_bounty_per_severity_low
    average_bounty_per_severity_medium
    average_bounty_per_severity_high
    average_bounty_per_severity_critical
    report_count_per_severity_low
    report_count_per_severity_medium
    report_count_per_severity_high
    report_count_per_severity_critical
  }
  bounty_table {
    low_label
    medium_label
    high_label
    critical_label
    description
    use_range
    bounty_table_rows(first: 100) {
      nodes {
        low
        medium
        high
        critical
        low_minimum
        medium_minimum
        high_minimum
        critical_minimum
        smart_rewards_start_at
        structured_scope {
          id
          asset_identifier
        }
        updated_at
      }
    }
    updated_at
  }
}

fragment ProfileMetrics on Team {
  currency
  offers_bounties
  average_bounty_lower_amount
  average_bounty_upper_amount
  top_bounty_lower_amount
  top_bounty_upper_amount
  formatted_total_bounties_paid_prefix
  formatted_total_bounties_paid_amount
  resolved_report_count
  formatted_bounties_paid_last_90_days
  hide_bounty_amounts
  reports_received_last_90_days
  last_report_resolved_at
  most_recent_sla_snapshot {
    first_response_time: average_time_to_first_program_response
    triage_time: average_time_to_report_triage
    bounty_time: average_time_to_bounty_awarded
    resolution_time: average_time_to_report_resolved
  }
  team_display_options {
    show_response_efficiency_indicator
    show_mean_first_response_time
    show_mean_report_triage_time
    show_mean_bounty_time
    show_mean_resolution_time
    show_top_bounties
    show_average_bounty
    show_total_bounties_paid
  }
}'''

PAGE_SIZE = 100

def get_all_programs():
  program_handles = []
  curr_page = 0
  while True:
    res = SESSION.post('https://hackerone.com/graphql', json={
      'query': PROGRAM_HANDLES_QUERY,
      'variables': {
        'size': PAGE_SIZE,
        'from': curr_page,
        'query': {},
        'filter': {
          'bool': {
            'filter': [{
              'bool': {
                'must_not': {'term': {'team_type': 'Engagements::Assessment'}},
                'should': [{'term': {'offers_bounties': True}}]
              }
            }]
          }
        },
        'sort': [{
          'field': 'launched_at',
          'direction': 'DESC'
        }],
        'post_filters': {
          'my_programs': False,
          'bookmarked': False,
          'campaign_teams': False
        }
      }
    }, allow_redirects=False)

    if res.status_code != 200:
      print(f'[!] Non-200 status code returned ({res.status_code}), stopping')
      print(res.content)
      break

    r_json = res.json()
    search = r_json.get('data', {}).get('opportunities_search', {})

    handles = [n['handle'] for n in search.get('nodes', [])]
    total_count = search.get('total_count', 0)

    if len(handles) == 0:
      print('[+] Got no more handles')
      break

    program_handles = list(set(program_handles + handles))

    if len(program_handles) >= total_count:
      print('[+] Got all handles')
      break

    curr_page += PAGE_SIZE

  return program_handles


def get_program_info(handle):
  res = SESSION.post('https://hackerone.com/graphql', json={
    'query': TEAMPROFILE_QUERY,
    'variables': {
      'handle': handle
    }
  })

  if res.status_code != 200:
    print(f'[!] Unable to fetch info for team handle: {handle}')
    return None

  r_json = res.json()
  return r_json

def check_auth():
    res = SESSION.post('https://hackerone.com/graphql', json={
      'query': ME_QUERY,
      'variables': {}
    })
    if res.status_code == 200:
      r_json = res.json()

      if not (me := r_json.get('data', {}).get('me', {})):
        return False

      return bool(me.get('id', None))

    return False


def check_program_data(handle, min_req, t_data=None):
  if t_data is None:
    p_info = get_program_info(handle)

    team_data = p_info.get('data', {}).get('team', {})
  else:
    team_data = t_data

  bounty_table_rows = (team_data.get('bounty_table', {}) or {}).get('bounty_table_rows', {}).get('nodes', [])

  if len(bounty_table_rows) == 0:
    return

  max_payout = team_data['top_bounty_upper_amount']
  max_bounty_crit = 0
  for bounty_table in bounty_table_rows:
    entry_max_bounty_crit = bounty_table['critical'] or bounty_table['critical_minimum'] or 0
    max_bounty_crit = max(max_bounty_crit, entry_max_bounty_crit)

  max_bounty_high = 0
  for bounty_table in bounty_table_rows:
    entry_max_bounty_high = bounty_table['high'] or bounty_table['high_minimum'] or 0
    max_bounty_high = max(max_bounty_high, entry_max_bounty_high)

  crit_percentage = {}

  findings = []
  if (metrics_snapshot := team_data['profile_metrics_snapshot']) is not None:
    if max_bounty_high > 0 and (avg_bounty_high := metrics_snapshot['average_bounty_per_severity_high']) is not None:
      avg_high_percent = avg_bounty_high / max_bounty_high
      if avg_high_percent >= 0.75:
        findings.append(f'Average high payout vs max high: {avg_high_percent:.0%}')

    if max_bounty_crit > 0 and (avg_bounty_crit := metrics_snapshot['average_bounty_per_severity_critical']) is not None:
      if (avg_crit_percent := avg_bounty_crit / max_bounty_crit) >= 0.75:
        findings.append(f'Average crit payout vs max crit: {avg_crit_percent:.0%}')

    low_report_count = metrics_snapshot['report_count_per_severity_low'] or 0
    medium_report_count = metrics_snapshot['report_count_per_severity_medium'] or 0
    high_report_count = metrics_snapshot['report_count_per_severity_high'] or 0
    critical_report_count = metrics_snapshot['report_count_per_severity_critical'] or 0
    total_report_metrics_count = low_report_count + medium_report_count + high_report_count + critical_report_count
    if total_report_metrics_count > 0:
      if (high_report_percent := high_report_count / total_report_metrics_count) >= 0.4:
        findings.append(f'Large percentage of high-severity reports: {high_report_percent:.2%} ({high_report_count} / {total_report_metrics_count})')

      if (critical_report_percent := critical_report_count / total_report_metrics_count) >= 0.4:
        findings.append(f'Large percentage of critical-severity reports: {critical_report_percent:.2%} ({critical_report_count} / {total_report_metrics_count})')

      if (high_and_crit_report_percent := (high_report_count + critical_report_count) / total_report_metrics_count) >= 0.4:
        findings.append(f'Large percentage of high + critical severity reports: {high_and_crit_report_percent:.2%} ({high_report_count + critical_report_count} / {total_report_metrics_count})')

      crit_percentage[handle] = critical_report_percent

  if max_payout is not None and max_bounty_crit > 0 and max_payout > max_bounty_crit:
    findings.append(f'Max payout > max crit: ${max_payout:,.2f} > ${max_bounty_crit:,.2f}')

  if len(findings) >= min_req:
    print(f'[{handle}]')
    print('\n'.join(f'  - {f}' for f in findings))


def check_positive(value):
    ivalue = int(value)
    if ivalue <= 0:
        raise argparse.ArgumentTypeError(f"{value} is an invalid positive int value")
    return ivalue

def main():
  global HOST_SESSION_COOKIE, CSRF_TOKEN, SESSION

  parser = argparse.ArgumentParser()
  parser.add_argument('-s', '--session', help='__Host-session cookie value from hackerone.com')
  parser.add_argument('-t', '--token', help='X-Csrf-Token value, you can fetch via: document.querySelector("meta[name=csrf-token]").getAttribute("content")')
  parser.add_argument('-f', '--force', action='store_true', help='Force re-fetch list of programs and program data')
  parser.add_argument('-m', '--min_req', default=1, help='Minimum number of identified features required to highlight a program in the output (default=1)', type=int)
  parser.add_argument('-c', '--check_auth', action='store_true', help='HackerOne auth check (for debug)')
  args = parser.parse_args()

  HOST_SESSION_COOKIE = args.session
  CSRF_TOKEN = args.token

  SESSION.cookies['__Host-session'] = HOST_SESSION_COOKIE
  SESSION.headers = {
    'Origin': 'https://hackerone.com',
    'x-csrf-token': CSRF_TOKEN
  }

  if args.min_req <= 0:
    print('[!] Error: min_req must be greater than zero')
    parser.print_help()
    return

  if args.check_auth:
    print(f'[+] Logged In: {check_auth()}')
    return

  # cache all program data locally if it doesnt exist
  all_prog_data = {}
  if not os.path.exists('program_data.json') or args.force:
    if not check_auth():
      print(f'[!] WARNING: Accessing HackerOne API unauthenticated. Only public programs will be fetched!')

    if not os.path.exists('programs.txt') or args.force:
      print('[+] Fetching program handles...')
      all_handles = get_all_programs()
      if len(all_handles) > 0:
        with open('programs.txt', 'w') as f:
          f.write('\n'.join(all_handles))
    else:
      all_handles = open('programs.txt').read().splitlines()

    print(f'Got {len(all_handles)} program handles, fetching and saving program data now...')
    for handle in tqdm(all_handles):
      p_info = get_program_info(handle)
      team_data = p_info.get('data', {}).get('team')
      if team_data is not None:
        all_prog_data[handle] = team_data
      else:
        print(f'[!] Uhoh, null team data returned for program: {handle}')

      with open('program_data.json', 'w') as f:
        f.write(json.dumps(all_prog_data, indent=4))
  else:
    all_prog_data = json.loads(open('program_data.json').read())

  for handle, team_data in sorted(all_prog_data.items()):
    check_program_data(handle, args.min_req, team_data)

if __name__ == '__main__':
  main()
```