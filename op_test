#!/usr/bin/env python3

import sys
import random
from argparse import ArgumentParser

def set_random_seed(seed_value):
    try:
        random.seed(seed_value, version=1)
    except:
        random.seed(seed_value)

def perform_job(job_description, start):
    job_id, duration, _ = job_description
    print(f'  [ time {start:.2f} ] Execute job {job_id} for {duration:.2f} secs (FINISH at {start + duration:.2f})')
    return start + duration

def manage_fifo(jobs_list, start):
    while jobs_list:
        job = jobs_list.pop(0)
        if start < job[2]:
            print(f'  [ time {start:.2f} ] Wait until {job[2]:.2f}')
            start = job[2]
        start = perform_job(job, start)
    return start

def process_round_robin(jobs_list, time_slice):
    from collections import deque
    queue = deque(jobs_list)
    current_time = 0
    while queue:
        job = queue.popleft()
        if current_time < job[2]:
            print(f'  [ time {current_time:.2f} ] Wait until {job[2]:.2f}')
            current_time = job[2]
        exec_time = min(job[1], time_slice)
        job[1] -= exec_time
        print(f'  [ time {current_time:.2f} ] Execute job {job[0]} for {exec_time:.2f} secs')
        current_time += exec_time
        if job[1] > 0:
            queue.append(job)
        else:
            print(f'  Job {job[0]} finished at time {current_time:.2f}')
    return current_time

def process_sjf(jobs_list):
    import heapq
    time_now = 0
    ready_jobs = []
    current_job = None

    while jobs_list or ready_jobs:
        while jobs_list and jobs_list[0][2] <= time_now:
            job = heapq.heappop(jobs_list)
            heapq.heappush(ready_jobs, (job[1], job[2], job[0]))

        if current_job and (not ready_jobs or current_job[0] <= ready_jobs[0][0]):
            duration_to_run = min(current_job[0], (ready_jobs[0][2] if ready_jobs else float('inf')) - time_now)
            print(f'  [ time {time_now:.2f} ] Execute job {current_job[2]} for {duration_to_run:.2f} secs')
            time_now += duration_to_run
            current_job = (current_job[0] - duration_to_run, current_job[1], current_job[2])
            if current_job[0] == 0:
                print(f'  Job {current_job[2]} finished at time {time_now:.2f}')
                current_job = None
        else:
            if current_job:
                heapq.heappush(ready_jobs, current_job)
            if ready_jobs:
                current_job = heapq.heappop(ready_jobs)
                continue

        if not current_job and jobs_list:
            next_arrival = jobs_list[0][2]
            print(f'  [ time {time_now:.2f} ] Wait until {next_arrival:.2f}')
            time_now = next_arrival

def main():
    parser = ArgumentParser()
    parser.add_argument("-s", "--seed", default=0, type=int, help="set random seed")
    parser.add_argument("-j", "--jobs", default=3, type=int, help="number of jobs in the system")
    parser.add_argument("-l", "--jlist", default="", type=str, help="comma-separated list of run times instead of random generation")
    parser.add_argument("-m", "--maxlen", default=10, type=int, help="maximum length of job")
    parser.add_argument("-p", "--policy", default="FIFO", type=str, help="scheduling policy: SJF, FIFO, RR")
    parser.add_argument("-q", "--quantum", default=1, type=int, help="time slice for RR policy")
    parser.add_argument("-c", action="store_true", default=False, help="compute answers")
    parser.add_argument("-a", "--arrival", default="", type=str, help="comma-separated list of arrival times")

    args = parser.parse_args()
    set_random_seed(args.seed)

    job_queue = []
    if args.jlist == '':
        for i in range(args.jobs):
            runtime = int(args.maxlen * random.random()) + 1
            arrival_time = 0 if args.arrival == '' else int(args.arrival.split(',')[i])
            job_queue.append([i, runtime, arrival_time])
    else:
        runtimes = args.jlist.split(',')
        for i, runtime in enumerate(runtimes):
            arrival_time = 0 if args.arrival == '' else int(args.arrival.split(',')[i])
            job_queue.append([i, float(runtime), arrival_time])

    job_queue.sort(key=lambda job: job[2])

    if args.c:
        print('** Solutions **\n')
        if args.policy == 'SJF':
            process_sjf(job_queue)
        elif args.policy == 'FIFO':
            manage_fifo(job_queue, 0)
        elif args.policy == 'RR':
            process_round_robin(job_queue, args.quantum)

if __name__ == "__main__":
    main()
